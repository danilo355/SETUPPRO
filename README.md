import os
import subprocess
import threading
import msvcrt
import tkinter as tk
from tkinter import messagebox, scrolledtext

def instaladores_pasta(pasta):
    arquivos = []
    for root, dirs, files in os.walk(pasta):
        for file in files:
            if file.lower().endswith(('.exe', '.msi')):
                arquivos.append(os.path.join(root, file))
    return arquivos

def instalar(instaladores, log_widget, btn_iniciar):
    btn_iniciar.config(state='disabled')
    cancelado = False

    log_widget.insert(tk.END, "🔄 Pressione DELETE a qualquer momento para cancelar a instalação.\n\n")
    log_widget.see(tk.END)

    for instalador in instaladores:
        if msvcrt.kbhit():
            tecla = msvcrt.getch()
            if tecla == b'\x2e':  # Delete
                log_widget.insert(tk.END, "\n❌ Instalação cancelada pelo usuário.\n")
                cancelado = True
                break

        nome = os.path.basename(instalador)
        log_widget.insert(tk.END, f"▶ Iniciando instalação: {nome}\n")
        log_widget.see(tk.END)

        if instalador.lower().endswith('.exe'):
            args = [instalador, '/S']
        elif instalador.lower().endswith('.msi'):
            args = ['msiexec', '/i', instalador, '/quiet', '/norestart']
        else:
            continue

        try:
            subprocess.run(args, check=True)
            log_widget.insert(tk.END, f"✅ {nome} instalado com sucesso.\n\n")
        except subprocess.CalledProcessError as e:
            log_widget.insert(tk.END, f"❌ Erro na instalação de {nome}: {e}\n\n")

        log_widget.see(tk.END)

    if not cancelado:
        log_widget.insert(tk.END, "\n🎉 Todas as instalações foram concluídas!\n")

    btn_iniciar.config(state='normal')

def iniciar_instalacao(btn_iniciar, log_widget):
    pasta = os.getcwd()
    instaladores = instaladores_pasta(pasta)
    if not instaladores:
        messagebox.showinfo("Aviso", "Nenhum instalador (.exe ou .msi) encontrado na pasta atual.")
        return

    thread = threading.Thread(target=instalar, args=(instaladores, log_widget, btn_iniciar))
    thread.start()

def criar_interface():
    root = tk.Tk()
    root.title("SetupPro - Instalador Automático")
    root.geometry("640x450")
    root.configure(bg="#1e1e2f")
    root.resizable(False, False)

    # Título
    titulo = tk.Label(root, text="🛠️ SetupPro", font=("Arial", 20, "bold"), bg="#1e1e2f", fg="white")
    titulo.pack(pady=(15, 5))

    subtitulo = tk.Label(root, text="Instalação silenciosa de pacotes .exe e .msi", font=("Arial", 12), bg="#1e1e2f", fg="#cccccc")
    subtitulo.pack()

    # Botão
    btn_iniciar = tk.Button(
        root, text="Iniciar Instalação", font=("Arial", 12, "bold"),
        width=20, height=2, bg="#4CAF50", fg="white", activebackground="#45a049"
    )
    btn_iniciar.pack(pady=15)

    # Log
    log_widget = scrolledtext.ScrolledText(
        root, width=75, height=20, bg="#282c34", fg="#dcdcdc", font=("Consolas", 10)
    )
    log_widget.pack(padx=10, pady=10)

    # Botão ativa função
    btn_iniciar.config(command=lambda: iniciar_instalacao(btn_iniciar, log_widget))

    root.mainloop()

if __name__ == "__main__":
    criar_interface()
