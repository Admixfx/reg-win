from tkinter import *
from tkinter import ttk, messagebox, simpledialog
import sqlite3
import hashlib

def is_valid_data(username, password):
    if len(username) < 4 or len(password) < 6:
        messagebox.showerror('Ошибка регистрации', 'Имя пользователя должно содержать не менее 4 символов, а пароль - не менее 6 символов.')
        return False
    return True

def save_user_data(username, password):
    hashed_password = hashlib.sha256(password.encode()).hexdigest()
    
    cursor.execute('INSERT INTO users (username, password) VALUES (?, ?)', (username, hashed_password))
    conn.commit()

def start_registration_window():
    registration_window = Toplevel(root)
    registration_window.title('Регистрация')
    registration_window.geometry('400x300')
    
    registration_label = ttk.Label(registration_window, text='Введите данные для регистрации', font='Arial 16 bold')
    registration_label.pack(pady=10)
    
    username_label = ttk.Label(registration_window, text='Имя пользователя:', font='Arial 12')
    username_label.pack(pady=5)
    
    username_entry = ttk.Entry(registration_window, font='Arial 12')
    username_entry.pack(pady=5)
    
    password_label = ttk.Label(registration_window, text='Пароль:', font='Arial 12')
    password_label.pack(pady=5)
    
    password_entry = ttk.Entry(registration_window, show='*', font='Arial 12')
    password_entry.pack(pady=5)
    
    register_button = ttk.Button(registration_window, text='Зарегистрироваться', command=lambda: process_registration(username_entry.get(), password_entry.get()))
    register_button.pack(pady=10)

def process_registration(username, password):
    if is_valid_data(username, password):
        save_user_data(username, password)
        messagebox.showinfo('Регистрация успешна!', 'Регистрация прошла успешно!')
    else:
        messagebox.showerror('Ошибка регистрации', 'Пожалуйста, проверьте данные.')

def login():
    username = username_entry.get()
    password = password_entry.get()

    hashed_password = hashlib.sha256(password.encode()).hexdigest()

    cursor.execute('SELECT * FROM users WHERE username=? AND password=?', (username, hashed_password))
    user = cursor.fetchone()

    if user:
        messagebox.showinfo('Авторизация успешна!', f'Добро пожаловать, {username}!')
    else:
        messagebox.showerror('Ошибка авторизации', 'Неверное имя пользователя или пароль.')

root = Tk()
root.title('Авторизация')
root.geometry('900x460')
root.resizable(width=False, height=False)

style = ttk.Style()
style.theme_use('default')

bg_color = '#d9d9d9'
root.configure(bg=bg_color)

main_label = ttk.Label(root, text='Авторизация', font='Arial 30 bold')
main_label.grid(row=0, column=0, columnspan=2, pady=(20, 40))

username_label = ttk.Label(root, text='Имя пользователя', font='Arial 22 bold')
username_label.grid(row=1, column=0, padx=20, pady=16, sticky=W)

username_entry = ttk.Entry(root, font='Arial 24', style='TEntry')
username_entry.grid(row=1, column=1, padx=20, pady=16, sticky=E)

password_label = ttk.Label(root, text='Пароль', font='Arial 22 bold')
password_label.grid(row=2, column=0, padx=20, pady=16, sticky=W)

password_entry = ttk.Entry(root, show='*', font='Arial 24', style='TEntry')
password_entry.grid(row=2, column=1, padx=20, pady=16, sticky=E)

send_btn = ttk.Button(root, text='Войти', command=login, style='TButton')
send_btn.grid(row=3, column=0, pady=(20, 40), sticky=W)

register_btn = ttk.Button(root, text='Зарегистрироваться', command=start_registration_window, style='TButton')
register_btn.grid(row=3, column=1, pady=(20, 40), sticky=E)


conn = sqlite3.connect('users.db')
cursor = conn.cursor()
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT NOT NULL,
        password TEXT NOT NULL
    )
''')
conn.commit()

root.mainloop()

conn.close()
