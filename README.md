import os
import customtkinter as ctk
import tkinter as tk
from tkinter import ttk, messagebox

def carregar_tema_dark():
    """Carrega o tema dark com tratamento de erros robusto"""
    theme_path = os.path.join(os.path.dirname(__file__), "tema_hospitalar.json")
    
    try:
        # Verificação de arquivo
        if not os.path.exists(theme_path):
            raise FileNotFoundError(f"Arquivo de tema não encontrado: {theme_path}")
        
        # Validação JSON
        with open(theme_path, "r", encoding="utf-8") as f:
            tema = json.load(f)
        
        # Verificação de estrutura
        if not all(key in tema for key in ["CTk", "CTkButton", "palette"]):
            raise ValueError("Estrutura do tema inválida")
        
        ctk.set_default_color_theme(theme_path)
        return True
    
    except Exception as e:
        messagebox.showwarning(
            "Tema Dark - Fallback",
            f"Tema personalizado não carregado:\n{str(e)}\nUsando tema dark padrão."
        )
        ctk.set_appearance_mode("dark")
        ctk.set_default_color_theme("dark-blue")
        return False

def verificar_usuario():
    usuario = campo_usuario.get()
    senha = campo_senha.get()

    if usuario == 'Daniel' and senha == '12345678':
        resultado_login.configure(text='Login bem sucedido', text_color='green')
        app.after(1000, lambda: [app.destroy(), menu_principal()])  # Fecha e chama o menu
    else:
        resultado_login.configure(text='Login incorreto', text_color='red')

app = ctk.CTk()
app.title('Login Consultório')
app.geometry('300x300')

label_usuario = ctk.CTkLabel(app, text='USUÁRIO:')
label_usuario.pack(pady=10)

campo_usuario = ctk.CTkEntry(app, placeholder_text='Digite seu usuário')
campo_usuario.pack(pady=10)

label_senha = ctk.CTkLabel(app, text='SENHA:')
label_senha.pack(pady=10)

campo_senha = ctk.CTkEntry(app, placeholder_text='Digite sua senha', show='*')
campo_senha.pack(pady=10)

botao_login = ctk.CTkButton(app, text='Login', command=verificar_usuario)
botao_login.pack(pady=10)

resultado_login = ctk.CTkLabel(app, text='')
resultado_login.pack(pady=10)

app.mainloop()


FILE_PATH = os.path.expanduser(r"~\Documents\SISTEMA\registros.txt")
AGENDAMENTO_PATH = os.path.expanduser(r"~\Documents\SISTEMA\agendamentos.txt")


def verificar_arquivo():
    diretorio = os.path.dirname(FILE_PATH)
    if diretorio and not os.path.exists(diretorio):
        os.makedirs(diretorio)
    if not os.path.isfile(FILE_PATH):
        with open(FILE_PATH, 'w') as f:
            pass

def verificar_agendamento():
    diretorio = os.path.dirname(AGENDAMENTO_PATH)
    if diretorio and not os.path.exists(diretorio):
        os.makedirs(diretorio)
    if not os.path.isfile(AGENDAMENTO_PATH):
        with open(AGENDAMENTO_PATH, 'w') as f:
            pass


# Função para salvar os dados no arquivo
def salvar_dados(tipo, dados):
    verificar_arquivo()
    with open(FILE_PATH, 'a') as f:
        f.write(f"{tipo};{';'.join(dados)}\n")
    messagebox.showinfo("Sucesso", f"{tipo} cadastrado com sucesso!")

# Função para realizar consultas
def janela_consulta(campo):
    janela = tk.Toplevel()
    janela.title(f"Consulta por {campo.capitalize()}")

    pesquisa_entry = tk.StringVar()
    resultados_frame = tk.Frame(janela)
    resultados_frame.grid(row=2, column=0, columnspan=2, pady=10)

    def realizar_consulta():
        valor = pesquisa_entry.get().strip().lower()
        resultados = []
        try:
            with open(FILE_PATH, 'r') as f:
                for linha in f:
                    dados = linha.strip().split(";")
                    if campo.lower() == "nome" and valor in dados[1].lower():
                        resultados.append(dados)
                    elif campo.lower() == "cpf" and valor == dados[5].lower():
                        resultados.append(dados)
        except FileNotFoundError:
            messagebox.showerror("Erro", "Arquivo de registros não encontrado!")
            return

        for widget in resultados_frame.winfo_children():
            widget.destroy()
        if resultados:
            for resultado in resultados:
                tk.Label(resultados_frame, text=" | ".join(resultado), anchor="w").pack(fill="x", padx=5, pady=2)
        else:
            tk.Label(resultados_frame, text="Nenhum resultado encontrado.", anchor="w", fg="red").pack(fill="x", padx=5, pady=2)

    ttk.Label(janela, text=f"Valor para Pesquisa ({campo.capitalize()}):").grid(row=0, column=0, pady=5, sticky=tk.W)
    ttk.Entry(janela, textvariable=pesquisa_entry, width=40).grid(row=0, column=1, pady=5)
    ttk.Button(janela, text="Consultar", command=realizar_consulta).grid(row=1, column=0, pady=10)
    ttk.Button(janela, text="Fechar", command=janela.destroy).grid(row=1, column=1, pady=10)

def janela_agendamento():
    janela = tk.Toplevel()
    janela.title("Agendamento")

    def confirmar_agendamento():
        nome = nome_entry.get()
        data = data_entry.get()
        horario = horario_entry.get()

        if not nome or not data or not horario:
            messagebox.showerror("Erro", "Preencha todos os campos!")
            return
        else:
            def salvar_agendamento(dados):
              with open(AGENDAMENTO_PATH, 'a') as f:
               f.write(";".join(dados) + "\n")

            salvar_agendamento([nome, data, horario])
            messagebox.showinfo("Sucesso", "Agendamento salvo com sucesso!")

        janela.destroy()

    ttk.Label(janela, text="Nome:").grid(row=0, column=0, pady=5, sticky=tk.W)
    nome_entry = ttk.Entry(janela)
    nome_entry.grid(row=0, column=1, pady=5)

    ttk.Label(janela, text="Data (dd/mm/aaaa):").grid(row=1, column=0, pady=5, sticky=tk.W)
    data_entry = ttk.Entry(janela)
    data_entry.grid(row=1, column=1, pady=5)

    ttk.Label(janela, text="Horário (hh:mm):").grid(row=2, column=0, pady=5, sticky=tk.W)
    horario_entry = ttk.Entry(janela)
    horario_entry.grid(row=2, column=1, pady=5)

    ttk.Button(janela, text="Confirmar", command=confirmar_agendamento).grid(row=3, column=0, pady=10)
    ttk.Button(janela, text="Cancelar", command=janela.destroy).grid(row=3, column=1, pady=10)


def visualizar_agendamentos_por_data():
    # Criar uma nova janela para exibir agendamentos
    janela = tk.Toplevel()
    janela.title("Visualizar Agendamentos por Data")

    # Entrada de data
    ttk.Label(janela, text="Digite a data (dd/mm/aaaa):").grid(row=0, column=0, padx=10, pady=10)
    data_entry = ttk.Entry(janela)
    data_entry.grid(row=0, column=1, padx=10, pady=10)

    # Frame para mostrar os resultados
    resultados_frame = tk.Frame(janela)
    resultados_frame.grid(row=2, column=0, columnspan=2, padx=10, pady=10)

    # Função para buscar e exibir os agendamentos
    def buscar_agendamentos():
        data_digitada = data_entry.get().strip()
        if not data_digitada:
            messagebox.showwarning("Atenção", "Digite uma data para consulta!")
            return

        if not os.path.isfile(AGENDAMENTO_PATH):
            messagebox.showerror("Erro", "Arquivo de agendamentos não encontrado!")
            return

        resultados = []
        with open(AGENDAMENTO_PATH, 'r') as f:
            for linha in f:
                nome, data, horario = linha.strip().split(";")
                if data == data_digitada:
                    resultados.append((nome, data, horario))

        # Limpar resultados anteriores
        for widget in resultados_frame.winfo_children():
            widget.destroy()

        # Exibir os novos resultados
        if resultados:
            for nome, data, horario in resultados:
                ttk.Label(resultados_frame, text=f"Nome: {nome} | Data: {data} | Horário: {horario}").pack(anchor='w')
        else:
            ttk.Label(resultados_frame, text="Nenhum agendamento encontrado.", foreground="red").pack(anchor='w')

    # Botões
    ttk.Button(janela, text="Buscar", command=buscar_agendamentos).grid(row=1, column=0, pady=10)
    ttk.Button(janela, text="Fechar", command=janela.destroy).grid(row=1, column=1, pady=10)



# Função para cadastro de dados
def janela_cadastro(tipo):
    janela = tk.Toplevel()
    janela.title(f"Cadastro de {tipo}")

    estado_var = tk.StringVar()
    sexo_var = tk.StringVar(value="Masculino")
    

    def salvar():
        nome = nome_entry.get()
        endereco = endereco_entry.get()
        cidade = cidade_entry.get()
        estado = estado_var.get()
        cpf = cpf_entry.get()
        email = email_entry.get()
        sexo = sexo_var.get()
        observacao = observacao_entry.get("1.0", tk.END).strip()

        if not all([nome, endereco, cidade, estado, cpf, email, sexo]):
            messagebox.showerror("Erro", "Preencha todos os campos obrigatórios!")
            return

        salvar_dados(tipo, [nome, endereco, cidade, estado, cpf, email, sexo, observacao])
        janela.destroy()

    ttk.Label(janela, text="Nome:").grid(row=0, column=0, sticky=tk.W, pady=5)
    nome_entry = ttk.Entry(janela)
    nome_entry.grid(row=0, column=1, pady=5)

    ttk.Label(janela, text="Endereço:").grid(row=1, column=0, sticky=tk.W, pady=5)
    endereco_entry = ttk.Entry(janela)
    endereco_entry.grid(row=1, column=1, pady=5)

    ttk.Label(janela, text="Cidade:").grid(row=2, column=0, sticky=tk.W, pady=5)
    cidade_entry = ttk.Entry(janela)
    cidade_entry.grid(row=2, column=1, pady=5)

    ttk.Label(janela, text="Estado:").grid(row=3, column=0, sticky=tk.W, pady=5)
    estados = ["AC", "AL", "AP", "AM", "BA", "CE", "DF", "ES", "GO", "MA", "MT", "MS", "MG", "PA", "PB", "PR", "PE", "PI", "RJ", "RN", "RS", "RO", "RR", "SC", "SP", "SE", "TO"]
    estado_combo = ttk.Combobox(janela, textvariable=estado_var, values=estados)
    estado_combo.grid(row=3, column=1, pady=5)

    ttk.Label(janela, text="CPF:").grid(row=4, column=0, sticky=tk.W, pady=5)
    cpf_entry = ttk.Entry(janela)
    cpf_entry.grid(row=4, column=1, pady=5)

    ttk.Label(janela, text="E-mail:").grid(row=5, column=0, sticky=tk.W, pady=5)
    email_entry = ttk.Entry(janela)
    email_entry.grid(row=5, column=1, pady=5)

    ttk.Label(janela, text="Sexo:").grid(row=6, column=0, sticky=tk.W, pady=5)
    sexo_frame = tk.Frame(janela)
    sexo_frame.grid(row=6, column=1, pady=5)
    tk.Radiobutton(sexo_frame, text="Masculino", variable=sexo_var, value="Masculino").pack(side=tk.LEFT)
    tk.Radiobutton(sexo_frame, text="Feminino", variable=sexo_var, value="Feminino").pack(side=tk.LEFT)
    tk.Radiobutton(sexo_frame, text="Outros", variable=sexo_var, value="Outros").pack(side=tk.LEFT)

    ttk.Label(janela, text="Observação:").grid(row=8, column=0, sticky=tk.W, pady=5)
    observacao_entry = tk.Text(janela, height=4, width=40)
    observacao_entry.grid(row=8, column=1, pady=5)

    ttk.Button(janela, text="Salvar", command=salvar).grid(row=9, column=0, pady=10)
    ttk.Button(janela, text="Cancelar", command=janela.destroy).grid(row=9, column=1, pady=10)

# Funções administrativas
def alterar_registro():
    messagebox.showinfo("Administração", "Alteração de registro ainda não implementada.")

def excluir_cadastro():
    messagebox.showinfo("Administração", "Exclusão de cadastro ainda não implementada.")

def gerar_relatorios():
    messagebox.showinfo("Administração", "Geração de relatórios ainda não implementada.")

# Menu principal
def menu_principal():
    janela = tk.Tk()
    janela.title("Sistema de Cadastro")

    menu = tk.Menu(janela)
    janela.config(menu=menu)

    cadastro_menu = tk.Menu(menu, tearoff=0)
    consulta_menu = tk.Menu(menu, tearoff=0)
    administrativo_menu = tk.Menu(menu, tearoff=0)
    agendamento_menu = tk.Menu(menu, tearoff=0)

    menu.add_cascade(label="Cadastro", menu=cadastro_menu)
    menu.add_cascade(label="Consulta", menu=consulta_menu)
    menu.add_cascade(label="Administrativo", menu=administrativo_menu)
    menu.add_cascade(label="Agendamento", command=lambda:janela_agendamento())
    menu.add_cascade(label="Visualizar Agendamentos", menu=agendamento_menu)


    agendamento_menu.add_command(label="Visualizar por Data", command=visualizar_agendamentos_por_data)

    cadastro_menu.add_command(label="Paciente", command=lambda: janela_cadastro("Paciente"))
    
    consulta_menu.add_command(label="Por Nome", command=lambda: janela_consulta("nome"))
    consulta_menu.add_command(label="Por CPF", command=lambda: janela_consulta("cpf"))
    

    administrativo_menu.add_command(label="Alterar Registro", command=alterar_registro)
    administrativo_menu.add_command(label="Excluir Cadastro", command=excluir_cadastro)
    administrativo_menu.add_command(label="Gerar Relatórios", command=gerar_relatorios)

    ttk.Label(janela, text="Bem-vindo ao Sistema de Cadastro").pack(pady=20)
    ttk.Button(janela, text="Sair", command=janela.destroy).pack(pady=10)

    janela.mainloop()

# Executar o sistema
if __name__ == "__main__":
    verificar_agendamento()
    verificar_arquivo()
    menu_principal()
