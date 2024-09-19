# **Campo Minado**

Um simples jogo de Campo Minado desenvolvido em Python utilizando a biblioteca Tkinter para a interface gráfica. O jogo permite que os jogadores marquem onde acreditam que há minas e descubram células no campo. O objetivo é revelar todas as células que não contêm minas sem detonar nenhuma delas.
Funcionalidades

    Geração do Campo: O campo de jogo é gerado com uma quantidade específica de minas espalhadas aleatoriamente.
    Interface Gráfica: Utiliza a biblioteca Tkinter para criar a interface gráfica com botões que representam as células do campo de jogo.
    Marcação de Minas: O botão direito do mouse é usado para marcar células onde o jogador acha que há uma mina.
    Revelação de Células: O botão esquerdo do mouse revela o conteúdo da célula clicada.
    Verificação de Vitória e Derrota: O jogo detecta quando o jogador ganha ou perde e exibe uma mensagem apropriada.
    Menu Principal: Permite iniciar o jogo em diferentes níveis de dificuldade (Fácil, Médio, Difícil) ou sair do jogo.

# **Requisitos**

    Python 3.x
    Biblioteca Tkinter (inclusa por padrão com a instalação do Python)

# **Como Jogar**

    Iniciar o Jogo:
        Selecione o nível de dificuldade desejado no menu principal.
        Clique no botão "Fácil", "Médio" ou "Difícil" para iniciar o jogo com o número correspondente de linhas, colunas e minas.

    Marcar Minas:
        Clique com o botão direito do mouse em uma célula para marcá-la com uma bandeira (⚑). Isso indica que você suspeita que há uma mina na célula.

    Revelar Células:
        Clique com o botão esquerdo do mouse em uma célula para revelá-la. Se a célula contém uma mina, o jogo terminará e as minas serão reveladas.
        Se a célula não contém minas, o número de minas adjacentes será exibido.

    Voltar ao Menu:
        Após uma vitória ou derrota, use o botão "Voltar ao Menu" para retornar ao menu principal e iniciar um novo jogo ou sair.

# **Código Fonte**

O código fonte do jogo está disponível no arquivo campo_minado.py.

 ```bash
   import tkinter as tk
import random

class CampoMinado:
    def __init__(self, root, linhas=10, colunas=10, bombas=15):
        self.root = root
        self.linhas = linhas
        self.colunas = colunas
        self.bombas = bombas
        self.campo = []
        self.botoes = []
        self.jogo_ativo = True
        self.gerar_campo()
        self.criar_interface()
        self.marcas = set()  # Para armazenar as marcas das bombas

    def gerar_campo(self):
        self.campo = [[0 for _ in range(self.colunas)] for _ in range(self.linhas)]
        for _ in range(self.bombas):
            while True:
                x = random.randint(0, self.linhas - 1)
                y = random.randint(0, self.colunas - 1)
                if self.campo[x][y] != "B":
                    self.campo[x][y] = "B"
                    break

        for i in range(self.linhas):
            for j in range(self.colunas):
                if self.campo[i][j] == "B":
                    continue
                contador_bombas = 0
                for dx in [-1, 0, 1]:
                    for dy in [-1, 0, 1]:
                        if 0 <= i + dx < self.linhas and 0 <= j + dy < self.colunas:
                            if self.campo[i + dx][j + dy] == "B":
                                contador_bombas += 1
                self.campo[i][j] = contador_bombas

    def criar_interface(self):
        for i in range(self.linhas):
            linha_botoes = []
            for j in range(self.colunas):
                botao = tk.Button(self.root, text="", width=3, height=2,
                                  command=lambda x=i, y=j: self.clicar_botao(x, y))
                botao.bind("<Button-3>", lambda e, x=i, y=j: self.marcar_botao(x, y))
                botao.grid(row=i, column=j)
                linha_botoes.append(botao)
            self.botoes.append(linha_botoes)

    def clicar_botao(self, x, y):
        if not self.jogo_ativo:
            return
        
        botao = self.botoes[x][y]
        
        # Verificar se o botão está marcado como uma bomba
        if (x, y) in self.marcas:
            return

        valor = self.campo[x][y]
        
        if valor == "B":
            botao.config(text="B", bg="red")
            self.game_over(vitoria=False)
        else:
            botao.config(text=str(valor), state="disabled", relief=tk.SUNKEN)
            if valor == 0:
                self.revelar_vazios(x, y)
            if self.verificar_vitoria():
                self.game_over(vitoria=True)

    def marcar_botao(self, x, y):
        if not self.jogo_ativo:
            return

        botao = self.botoes[x][y]
        if botao.cget("text") == "":  # Se o botão está vazio
            botao.config(text="⚑")
            self.marcas.add((x, y))
        elif botao.cget("text") == "⚑":  # Se o botão já tem uma marca
            botao.config(text="")
            self.marcas.remove((x, y))

    def revelar_vazios(self, x, y):
        for dx in [-1, 0, 1]:
            for dy in [-1, 0, 1]:
                if 0 <= x + dx < self.linhas and 0 <= y + dy < self.colunas:
                    botao = self.botoes[x + dx][y + dy]
                    if botao['state'] == 'normal':
                        botao.config(text=str(self.campo[x + dx][y + dy]), state="disabled", relief=tk.SUNKEN)
                        if self.campo[x + dx][y + dy] == 0:
                            self.revelar_vazios(x + dx, y + dy)

    def verificar_vitoria(self):
        for i in range(self.linhas):
            for j in range(self.colunas):
                if self.campo[i][j] != "B" and self.botoes[i][j]['state'] == 'normal':
                    return False
        return True

    def game_over(self, vitoria):
        self.jogo_ativo = False
        for i in range(self.linhas):
            for j in range(self.colunas):
                if self.campo[i][j] == "B":
                    self.botoes[i][j].config(text="B", bg="red")
        self.mostrar_resultado(vitoria)

    def mostrar_resultado(self, vitoria):
        resultado_frame = tk.Frame(self.root)
        resultado_frame.grid(row=0, column=0, columnspan=self.colunas, rowspan=self.linhas)

        resultado_titulo = "Você venceu!" if vitoria else "Você perdeu!"
        resultado_label = tk.Label(resultado_frame, text=resultado_titulo, font=("Helvetica", 16))
        resultado_label.grid(row=0, column=0, padx=10, pady=10)

        reiniciar_botao = tk.Button(resultado_frame, text="Voltar ao Menu", command=self.voltar_menu)
        reiniciar_botao.grid(row=1, column=0, padx=10, pady=10)

    def voltar_menu(self):
        self.root.destroy()
        root = tk.Tk()
        MenuPrincipal(root)
        root.mainloop()

class MenuPrincipal:
    def __init__(self, root):
        self.root = root
        self.root.title("Campo Minado - Menu Principal")
        self.frame = tk.Frame(self.root)
        self.frame.grid(row=0, column=0)

        self.titulo = tk.Label(self.frame, text="Bem-vindo ao Campo Minado", font=("Helvetica", 16))
        self.titulo.grid(row=0, column=0, padx=10, pady=10)

        self.botao_facil = tk.Button(self.frame, text="Fácil", command=self.iniciar_jogo_facil)
        self.botao_facil.grid(row=1, column=0, padx=10, pady=5)

        self.botao_medio = tk.Button(self.frame, text="Médio", command=self.iniciar_jogo_medio)
        self.botao_medio.grid(row=2, column=0, padx=10, pady=5)

        self.botao_dificil = tk.Button(self.frame, text="Difícil", command=self.iniciar_jogo_dificil)
        self.botao_dificil.grid(row=3, column=0, padx=10, pady=5)

        self.botao_sair = tk.Button(self.frame, text="Sair", command=self.root.quit)
        self.botao_sair.grid(row=4, column=0, padx=10, pady=20)

    def iniciar_jogo(self, linhas, colunas, bombas):
        self.frame.destroy()
        app = CampoMinado(self.root, linhas, colunas, bombas)

    def iniciar_jogo_facil(self):
        self.iniciar_jogo(8, 8, 10)

    def iniciar_jogo_medio(self):
        self.iniciar_jogo(16, 16, 40)

    def iniciar_jogo_dificil(self):
        self.iniciar_jogo(24, 24, 99)

if __name__ == "__main__":
    root = tk.Tk()
    MenuPrincipal(root)
    root.mainloop()

   ```

# **Contribuições**

Contribuições são bem-vindas! Se você deseja contribuir com o projeto, por favor, abra uma issue ou envie um pull request.

# **Licença**

Este projeto está licenciado sob a licença MIT