# Final_progect.py

import tkinter as tk
from tkinter import simpledialog
import random

# ==================== НАЛАШТУВАННЯ ====================

TIME_LIMIT = 300

BG = "#101018"
PANEL = "#191925"
BUTTON = "#29293B"
TEXT = "#E5E5F0"
GREEN = "#80FFB0"
RED = "#FF666F"
PURPLE = "#C5A3FF"
YELLOW = "#FFE28A"

# ==================== ЗМІННІ ГРИ ====================

time_left = TIME_LIMIT
danger = 0
game_running = False
game_over = False

inventory = []
clues = set()
opened = set()

secret_found = False
event_counter = 0
door_attempts = 0

# ==================== ВІКНО ====================

root = tk.Tk()
root.title("НЕ ДИВИСЬ НАЗАД")
root.geometry("760x760")
root.resizable(False, False)
root.configure(bg=BG)

# ==================== ІНТЕРФЕЙС ====================

title_label = tk.Label(
    root,
    text="НЕ ДИВИСЬ НАЗАД",
    font=("Consolas", 25, "bold"),
    fg=GREEN,
    bg=BG
)
title_label.pack(pady=(15, 3))

subtitle_label = tk.Label(
    root,
    text="ЗАКРИТИЙ СЕКТОР // НІЧ 01",
    font=("Consolas", 10),
    fg=PURPLE,
    bg=BG
)
subtitle_label.pack()

timer_label = tk.Label(
    root,
    text="ЧАС: 05:00",
    font=("Consolas", 18, "bold"),
    fg=YELLOW,
    bg=BG
)
timer_label.pack(pady=(10, 3))

danger_label = tk.Label(
    root,
    text="НЕБЕЗПЕКА: 0%",
    font=("Consolas", 13, "bold"),
    fg=GREEN,
    bg=BG
)
danger_label.pack(pady=3)

danger_bar = tk.Canvas(
    root,
    width=350,
    height=12,
    bg="#30303D",
    highlightthickness=0
)
danger_bar.pack(pady=4)

danger_fill = danger_bar.create_rectangle(
    0, 0, 0, 12,
    fill=GREEN,
    outline=""
)

status_label = tk.Label(
    root,
    text="Натисни «ПОЧАТИ», щоб увійти до кімнати.",
    font=("Consolas", 10),
    fg=TEXT,
    bg=BG,
    wraplength=690
)
status_label.pack(pady=5)

story_frame = tk.Frame(
    root,
    bg=PANEL,
    highlightbackground="#35354B",
    highlightthickness=1
)
story_frame.pack(
    padx=25,
    pady=10,
    fill="both",
    expand=True
)

story_title = tk.Label(
    story_frame,
    text="// СИСТЕМНИЙ ЖУРНАЛ",
    font=("Consolas", 11, "bold"),
    fg=GREEN,
    bg=PANEL,
    anchor="w"
)
story_title.pack(fill="x", padx=12, pady=(10, 3))

story_text = tk.Text(
    story_frame,
    bg=PANEL,
    fg=TEXT,
    font=("Consolas", 10),
    wrap="word",
    relief="flat",
    height=12,
    padx=12,
    pady=8,
    state="disabled",
    insertbackground=GREEN
)
story_text.pack(fill="both", expand=True, padx=5, pady=5)

story_text.tag_configure("green", foreground=GREEN)
story_text.tag_configure("red", foreground=RED)
story_text.tag_configure("purple", foreground=PURPLE)
story_text.tag_configure("yellow", foreground=YELLOW)

inventory_label = tk.Label(
    root,
    text="ІНВЕНТАР: порожньо",
    font=("Consolas", 10),
    fg=YELLOW,
    bg=BG,
    wraplength=700
)
inventory_label.pack(pady=5)

button_frame = tk.Frame(root, bg=BG)
button_frame.pack(padx=15, pady=(5, 12))

buttons = {}


def make_button(name, text, command, row, column):
    button = tk.Button(
        button_frame,
        text=text,
        command=command,
        font=("Consolas", 10, "bold"),
        bg=BUTTON,
        fg=TEXT,
        activebackground="#41415B",
        activeforeground=GREEN,
        disabledforeground="#777788",
        relief="flat",
        bd=0,
        width=17,
        height=2,
        cursor="hand2"
    )
    button.grid(row=row, column=column, padx=5, pady=5)
    buttons[name] = button


# ==================== СИСТЕМНІ ФУНКЦІЇ ====================

def log(message, tag=None):
    story_text.config(state="normal")
    story_text.insert("end", message + "\n\n", tag)
    story_text.see("end")
    story_text.config(state="disabled")


def update_inventory():
    if inventory:
        text = "ІНВЕНТАР: " + " | ".join(inventory)
    else:
        text = "ІНВЕНТАР: порожньо"

    inventory_label.config(text=text)


def update_status(message):
    status_label.config(text=message)


def set_buttons_enabled(enabled):
    for button in buttons.values():
        button.config(
            state="normal" if enabled else "disabled"
        )


def add_item(item):
    if item not in inventory:
        inventory.append(item)
        update_inventory()
        log("ЗНАЙДЕНО: " + item, "yellow")


def end_game(title, message, color):
    global game_running, game_over

    if game_over:
        return

    game_running = False
    game_over = True

    set_buttons_enabled(False)
    start_button.config(state="disabled")
    restart_button.config(state="normal")

    update_status(title)

    log("=" * 35, color)
    log(title, color)
    log(message, color)
    log("=" * 35, color)


# ==================== МЕХАНІКА НЕБЕЗПЕКИ ====================

events = [
    "Лампа блимає. У кутку кімнати ніби хтось стоїть.",
    "На екрані з'являється повідомлення: «Я БАЧУ ТЕБЕ».",
    "За дверима лунають три повільні удари.",
    "Монітор самостійно відкриває файл із написом «БЛИЖЧЕ».",
    "Світло гасне на мить. Коли воно повертається, стілець стоїть інакше.",
    "Ти чуєш кроки за спиною. Вони зупиняються разом із тобою."
]


def update_danger_bar():
    danger_label.config(
        text=f"НЕБЕЗПЕКА: {danger}%",
        fg=GREEN if danger < 40 else YELLOW if danger < 75 else RED
    )

    width = int(350 * danger / 100)

    danger_bar.coords(
        danger_fill,
        0, 0, width, 12
    )

    color = GREEN if danger < 40 else YELLOW if danger < 75 else RED
    danger_bar.itemconfig(danger_fill, fill=color)


def random_event():
    global event_counter, danger

    if not game_running:
        return

    event_counter += 1

    # Кожна дія наближає істоту
    danger += random.randint(4, 9)

    # Імовірність містичної події зростає
    chance = min(0.30 + event_counter * 0.05, 0.75)

    if random.random() < chance:
        log("!!! НЕВІДОМА ПОДІЯ !!!", "red")
        log(random.choice(events), "purple")
        danger += random.randint(8, 18)

    danger = min(danger, 100)
    update_danger_bar()

    if danger >= 100:
        end_game(
            "ПОРАЗКА: ВОНО ТЕБЕ ЗНАЙШЛО",
            "Світло гасне.\n\n"
            "На моніторі з'являється напис:\n"
            "«ТИ ЗАНАДТО ДОВГО ТУТ БУВ».\n\n"
            "Істота знайшла тебе. Гра завершена.\n"
            "Натисни «ЗАНОВО», щоб спробувати ще раз.",
            "red"
        )
        return

    if danger >= 75:
        update_status("КРИТИЧНА НЕБЕЗПЕКА! Істота зовсім близько!")
    elif danger >= 40:
        update_status("Ти чуєш кроки. Небезпека зростає.")


# ==================== ТАЙМЕР ====================

def update_timer():
    global time_left

    if not game_running:
        return

    minutes = time_left // 60
    seconds = time_left % 60

    timer_label.config(
        text=f"ЧАС: {minutes:02d}:{seconds:02d}",
        fg=RED if time_left <= 60 else YELLOW
    )

    if time_left <= 0:
        end_game(
            "ПОРАЗКА: ЧАС ВИЧЕРПАНО",
            "Комп'ютер гасне. Двері повільно відчиняються.\n\n"
            "За ними немає коридору — лише темрява.\n\n"
            "На моніторі з'являється повідомлення:\n"
            "«СЕКТОР ЗАКРИТО. ОБ'ЄКТ ЗАЛИШИВСЯ ВСЕРЕДИНІ».\n\n"
            "Спробуй ще раз.",
            "red"
        )
        return

    time_left -= 1
    root.after(1000, update_timer)


# ==================== ДОСЛІДЖЕННЯ КІМНАТИ ====================

def explore_computer():
    global secret_found

    if not game_running:
        return

    if "computer" not in opened:
        opened.add("computer")

        log(
            "КОМП'ЮТЕР\n"
            "На екрані три файли:\n"
            "READ_ME, CAMERA_01, DELETED_LOG",
            "green"
        )

        log(
            "У файлі READ_ME написано:\n"
            "«Код складається з трьох цифр.\n"
            "Перша — кількість очей у людини.\n"
            "Друга — кількість кутів у трикутника.\n"
            "Третя — кількість дверей у цій кімнаті»."
        )

        clues.add("code_hint")
        log("Підказку до коду збережено.", "yellow")

    elif "Ключ" in inventory and not secret_found:
        secret_found = True

        log(
            "На екрані відкривається SECRET.txt.\n\n"
            "«Ця кімната — експеримент.\n"
            "Єдиний вихід не завжди веде назовні».\n\n"
            "Тепер доступна таємна кінцівка.",
            "purple"
        )

    elif secret_found:
        log(
            "SECRET.txt: «Не кожні двері ведуть додому».",
            "purple"
        )

    else:
        log(
            "ДОСТУП ОБМЕЖЕНО.\n"
            "На екрані написано: «ЗНАЙДИ КЛЮЧ».",
            "red"
        )

    random_event()


def explore_desk():
    if not game_running:
        return

    if "desk" not in opened:
        opened.add("desk")

        log(
            "СТІЛ\n"
            "Під паперами ти знаходиш записку:\n\n"
            "«Не вір екрану.\n"
            "Ключ там, де ховають речі, "
            "які не хочуть бачити»."
        )

        clues.add("note")
        log("Записку додано до підказок.", "yellow")

    else:
        log("Ти переглянув папери. Більше нічого немає.")

    random_event()


def explore_wardrobe():
    if not game_running:
        return

    if "wardrobe" not in opened:
        opened.add("wardrobe")

        log(
            "ШАФА\n"
            "Дверцята скриплять. У кишені старого пальта "
            "лежить металевий ключ.",
            "yellow"
        )

        add_item("Ключ")

        log(
            "На внутрішній стороні дверцят помітні подряпини. "
            "Хтось уже намагався вибратися звідси."
        )

    else:
        log(
            "Шафа порожня. Але зсередини чути тихий стукіт.",
            "purple"
        )

    random_event()


def explore_window():
    if not game_running:
        return

    if "window" not in opened:
        opened.add("window")

        log(
            "ВІКНО\n"
            "За склом не місто, а нескінченний темний коридор."
        )

        log(
            "На запітнілому склі з'являються цифри: 2 - 3 - 1.",
            "yellow"
        )

        clues.add("strange_numbers")

    else:
        log(
            "У відображенні кімнати немає тебе. "
            "Лише темна постать за твоїм місцем.",
            "purple"
        )

    random_event()


# ==================== ДВЕРІ ТА КОД ====================

def explore_door():
    global door_attempts

    if not game_running:
        return

    if "Ключ" not in inventory:
        log(
            "ДВЕРІ ЗАМКНЕНІ.\n"
            "Електронний замок блимає червоним. "
            "Тобі потрібен ключ.",
            "red"
        )
        random_event()
        return

    if "code_hint" not in clues:
        log(
            "Ключ підходить до замка, але потрібен тризначний код.\n"
            "Спочатку досліди комп'ютер.",
            "yellow"
        )
        random_event()
        return

    answer = simpledialog.askstring(
        "Електронний замок",
        "Введи тризначний код:"
    )

    if not game_running or answer is None:
        return

    if answer.strip() == "231":
        if secret_found:
            end_game(
                "ТАЄМНА КІНЦІВКА: ЗА МЕЖАМИ",
                "Замок клацає. Двері відчиняються.\n\n"
                "За ними світанок. Ти виходиш назовні.\n\n"
                "На екрані старого комп'ютера з'являється:\n"
                "«ОБ'ЄКТ ВИЙШОВ ЗА МЕЖІ ЕКСПЕРИМЕНТУ».\n\n"
                "Ти знайшов таємну кінцівку!",
                "purple"
            )
        else:
            end_game(
                "КІНЦІВКА: ВТЕЧА",
                "Замок клацає. Двері відчиняються!\n\n"
                "Ти біжиш темним коридором, не озираючись.\n"
                "Позаду лунає голос: «ТИ ЩЕ ПОВЕРНЕШСЯ...»\n\n"
                "Ти врятувався, але таємниця залишилася.",
                "green"
            )

    else:
        door_attempts += 1

        log(
            "НЕПРАВИЛЬНИЙ КОД!\n"
            "Замок видає різкий сигнал. "
            "У кімнаті стає холодніше.",
            "red"
        )

        if door_attempts >= 2:
            log(
                "На дисплеї з'являється напис:\n"
                "«МИ ЗНАЄМО, ЩО ТИ РОБИШ».",
                "purple"
            )

        random_event()


# ==================== СТАРТ І ПЕРЕЗАПУСК ====================

def start_game():
    global time_left, danger, game_running, game_over
    global inventory, clues, opened
    global secret_found, event_counter, door_attempts

    time_left = TIME_LIMIT
    danger = 0
    game_running = True
    game_over = False

    inventory = []
    clues = set()
    opened = set()

    secret_found = False
    event_counter = 0
    door_attempts = 0

    story_text.config(state="normal")
    story_text.delete("1.0", "end")
    story_text.config(state="disabled")

    update_inventory()
    update_danger_bar()

    set_buttons_enabled(True)
    start_button.config(state="disabled")
    restart_button.config(state="normal")

    update_status("Знайди вихід до того, як сплине час.")

    log("23:47 // СИСТЕМА ЗАПУЩЕНА", "green")

    log(
        "Ти прокидаєшся в незнайомій кімнаті. "
        "Двері замкнені, вікно не відчиняється."
    )

    log(
        "На комп'ютері з'являється повідомлення:\n\n"
        "«Ти маєш п'ять хвилин.\n"
        "Знайди ключ. Розгадай код.\n"
        "І заради всього святого — НЕ ДИВИСЬ НАЗАД».",
        "red"
    )

    timer_label.config(text="ЧАС: 05:00", fg=YELLOW)

    root.after(1000, update_timer)


def restart_game():
    start_game()


# ==================== КНОПКИ ====================

make_button("computer", "КОМП'ЮТЕР", explore_computer, 0, 0)
make_button("desk", "СТІЛ", explore_desk, 0, 1)
make_button("wardrobe", "ШАФА", explore_wardrobe, 0, 2)
make_button("window", "ВІКНО", explore_window, 1, 0)
make_button("door", "ДВЕРІ", explore_door, 1, 1)

start_button = tk.Button(
    button_frame,
    text="ПОЧАТИ",
    command=start_game,
    font=("Consolas", 10, "bold"),
    bg="#28563D",
    fg="white",
    activebackground="#367C52",
    relief="flat",
    width=17,
    height=2,
    cursor="hand2"
)
start_button.grid(row=2, column=0, padx=5, pady=5)

restart_button = tk.Button(
    button_frame,
    text="ЗАНОВО",
    command=restart_game,
    font=("Consolas", 10, "bold"),
    bg="#493052",
    fg="white",
    activebackground="#68416F",
    relief="flat",
    width=17,
    height=2,
    cursor="hand2"
)
restart_button.grid(row=2, column=1, padx=5, pady=5)

set_buttons_enabled(False)
restart_button.config(state="disabled")


def enter_key(event):
    if not game_running and not game_over:
        start_game()


root.bind("<Return>", enter_key)

root.mainloop()
