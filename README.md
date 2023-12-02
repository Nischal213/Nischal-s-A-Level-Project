import random
import string
import pymysql
from tkinter import *
db = pymysql.connect(
    host='localhost',
    user='root',
    password='root',
    database='game_db')
cursor = db.cursor()
# ---------------------------------------------------------------------------------- (Allowing Access to db)
attempt = 1
holding_username = ''


def clear_box(box_type, button_name):
    box_type['text'] = ''
    button_name['state'] = 'normal'


def center_window(window_name=Tk):
    screen_width = int((window_name.winfo_screenwidth()) / 2 - 375)
    screen_height = int((window_name.winfo_screenheight()) / 2 - 250)
    window_name.geometry(f'700x500+{screen_width}+{screen_height}')


window_destroyed = False


def authenticate_user():
    def registration_page():
        global window_destroyed
        window.destroy()
        window_destroyed = True
        root = Tk()
        center_window(root)
        change = False
        question_number = 0

        def change_questions():
            global hold_first_name, hold_password, hold_last_name
            nonlocal change, question_number
            if question_number == 0:
                first_name = place_holder_box.get()
                last_name = place_holder_box2.get()
                if first_name == '' or last_name == '':
                    change = True
                    error_string = 'Cannot be blank'
                elif not (first_name.isalpha()) or not (last_name.isalpha()):
                    change = True
                    error_string = 'Name cannot contain special chars'
                if change:
                    button['state'] = 'disabled'
                    error_message['text'] = error_string
                    root.after(2000, clear_box, error_message, button)
                    change = False
                else:
                    question_number += 1
                    hold_first_name, hold_last_name = first_name, last_name
                    place_holder_box.delete(0, END)
                    place_holder_box2.delete(0, END)
                    button['state'] = 'disabled'
                    root.after(100, clear_box, error_message, button)
            if question_number == 1:
                place_holder_label['text'] = 'Password'
                place_holder_label2['text'] = 'Confirm Password'
                if button['state'] == 'normal':
                    password = place_holder_box.get()
                    conf_password = place_holder_box2.get()
                    if not (password == conf_password):
                        error_string = 'Passwords not matching'
                        change = True
                    if password == '' or conf_password == '':
                        error_string = 'Password cannot be blank'
                        change = True
                    if change:
                        error_message['text'] = error_string
                        button['state'] = 'disabled'
                        root.after(2000, clear_box, error_message, button)
                        change = False
                    else:
                        question_number += 1
                        hold_password = password
                        place_holder_box.delete(0, END)
                        place_holder_box2.delete(0, END)
                        button['state'] = 'disabled'
                        root.after(100, clear_box, error_message, button)
            if question_number == 2:
                def has_special_chars(str):
                    for i in str:
                        if i in string.punctuation:
                            return True
                    else:
                        return False

                # Go over this with huzaifa to check for any improvements
                def email_validator(str):
                    if has_special_chars(str):
                        if str.count('.') != 0 and str.count('@') == 1:
                            before_at_symbol = len(str[:str.index('@')])
                            after_at_symbol = len(str[str.index('@') + 1:])
                            if before_at_symbol > 0 and after_at_symbol > 0:
                                return True
                            else:
                                return False
                        else:
                            return False
                    else:
                        return False

                def new_clear_box():
                    username_error_box['text'] = ''
                    email_error_box['text'] = ''
                    button['state'] = 'normal'
                place_holder_label['text'] = 'Username'
                place_holder_label2['text'] = 'Email'
                username_error_string = ''
                email_error_string = ''
                if button['state'] == 'normal':
                    username = place_holder_box.get()
                    email = place_holder_box2.get()
                    cursor.execute(
                        f'SELECT username FROM user_info WHERE username = "{username}"')
                    username_row = cursor.rowcount
                    cursor.execute(
                        f'SELECT email FROM user_info WHERE email = "{email}" ')
                    email_row = cursor.rowcount
                    if username_row != 0:
                        change = True
                        username_error_string = 'Username already taken'
                    elif len(username) > 20 or len(username) < 3:
                        change = True
                        username_error_string = 'Username can only be 3-20 chars long'
                    elif has_special_chars(username):
                        change = True
                        username_error_string = 'Username must not have special chars'
                    elif len([i for i in username if i in string.digits]) == len(username):
                        change = True
                        username_error_string = 'Username may contain private info'
                    if not (email_validator(email)):
                        change = True
                        email_error_string = 'Email does not exist'
                    elif email_row != 0:
                        change = True
                        email_error_string = 'Email already taken'
                    if change:
                        username_error_box['text'] = username_error_string
                        email_error_box['text'] = email_error_string
                        button['state'] = 'disabled'
                        root.after(2000, new_clear_box)
                        change = False
                    else:
                        global holding_username
                        holding_username = username
                        button['state'] = 'disabled'
                        cursor.execute(
                            f'INSERT INTO user_info (first_name,last_name,username,pass_word,email,best_point) VALUES ("{hold_first_name}","{hold_last_name}","{username}","{hold_password}","{email}",0)')
                        db.commit()
                        root.destroy()
                        direct_to_login()

        def previous_page():
            root.destroy()
            authenticate_user()
        label = Label(text='Register Page', font=24)
        label.pack()
        place_holder_label = Label(text='First name:', font=24)
        place_holder_label.pack(pady=10)
        place_holder_box = Entry()
        place_holder_box.pack()
        place_holder_label2 = Label(text='Last name:', font=24)
        place_holder_label2.pack(pady=10)
        place_holder_box2 = Entry()
        place_holder_box2.pack()
        button = Button(text='Enter', command=change_questions,
                        font=('Microsoft JhengHei UI', 13))
        button.pack(pady=10)
        error_message = Message(text='', fg='red', font=24, width=450)
        error_message.pack()
        username_error_box = Message(text='', font=24, fg='red', width=550)
        username_error_box.place(x=420, y=64)
        email_error_box = Message(text='', font=24, fg='red', width=550)
        email_error_box.place(x=420, y=128)
        back_to_register = Button(
            text='← Back', command=previous_page, font=('Microsoft JhengHei UI', 13))
        back_to_register.place(x=0, y=0)
        root.resizable(False, False)
        root.protocol('WM_DELETE_WINDOW', False)
        root.mainloop()

    def direct_to_login():
        real = Tk()
        center_window(real)

        def to_login_page():
            real.destroy()
            login_page()
        label = Label(real, text='An account has been created!', font=24)
        label.pack()
        button = Button(real, text='Go to login page', command=to_login_page, font=(
            'Microsoft JhengHei UI', 13), width=20)
        button.pack(pady=20)
        real.resizable(False, False)
        real.protocol('WM_DELETE_WINDOW', False)
        real.mainloop()

    def login_page():
        if not (window_destroyed):
            window.destroy()

        def account_verification():
            global attempt
            pass_flag = True
            error_string = ''
            username = username_entry.get()
            password = password_entry.get()
            cursor.execute(
                f'SELECT username, pass_word FROM user_info WHERE username = "{username}" AND pass_word = "{password}"')
            if cursor.rowcount == 0:
                error_string = f'Invalid password or username! Attempt: {attempt}'
                pass_flag = False
            if not (pass_flag) and error_string:
                if attempt == 6:
                    attempt = 0
                    error_message['text'] = 'Too many failed attempts in a row, Temporarily on lockdown for 30 seconds'
                    check_button['state'] = 'disabled'
                    master.after(30000, clear_box, error_message, check_button)
                else:
                    check_button['state'] = 'disabled'
                    error_message['text'] = error_string
                    master.after(2000, clear_box, error_message, check_button)
                    attempt += 1
            else:
                global holding_username
                holding_username = username
                master.destroy()

        def forgot_account():  # Make sure to come back to this as this needs to be worked on
            master.destroy()

            def forgot_account_verification():
                def custom_clear_box():
                    username_box['text'] = ''
                    email_box['text'] = ''
                    button['state'] = 'normal'
                modify = False
                username = username_entry.get()
                cursor.execute(
                    f'SELECT username FROM user_info WHERE username = "{username}"')
                if cursor.rowcount == 0:
                    username_box['text'] = 'Username does not exist'
                    modify = True
                email = email_entry.get()
                cursor.execute(
                    f'SELECT email FROM user_info WHERE email = "{email}" and username = "{username}"')
                if cursor.rowcount == 0:
                    email_box['text'] = 'Email does not exist'
                    modify = True
                if modify:
                    button['state'] = 'disabled'
                    new_master.after(2000, custom_clear_box)
                else:
                    new_master.destroy()

            def previous_page():
                new_master.destroy()
                authenticate_user()
            new_master = Tk()
            center_window(new_master)
            label = Label(text='Forgot Page', font=24)
            label.pack()
            username_label = Label(text='Username', font=24)
            username_label.pack(pady=10)
            username_entry = Entry()
            username_entry.pack()
            username_box = Message(text='', font=24, fg='red', width=350)
            username_box.pack()
            email_label = Label(text='Email', font=24)
            email_label.pack()
            email_entry = Entry()
            email_entry.pack()
            email_box = Message(text='', font=24, fg='red', width=350)
            email_box.pack()
            button = Button(text='Enter', font=(
                'Microsoft JhengHei UI', 13), command=forgot_account_verification)
            button.pack(pady=10)
            back_to_login = Button(
                text='← Back', command=previous_page, font=('Microsoft JhengHei UI', 13))
            back_to_login.place(x=0, y=0)
            new_master.resizable(False, False)
            new_master.protocol('WM_DELETE_WINDOW', False)
            new_master.mainloop()

        def previous_page():
            master.destroy()
            authenticate_user()
        master = Tk()
        center_window(master)
        label = Label(text='Login Page', font=24)
        label.pack()
        username_label = Label(text='Username:', font=24)
        username_label.pack(pady=10)
        username_entry = Entry()
        username_entry.pack()
        password_label = Label(master, text='Password:', font=24)
        password_label.pack(pady=10)
        password_entry = Entry()
        password_entry.pack()
        check_button = Button(text='Enter', command=account_verification, font=(
            'Microsoft JhengHei UI', 13), width=10)
        check_button.pack(pady=10)
        error_message = Message(text='', font=24, fg='red', width=550)
        error_message.pack()
        back_to_register = Button(
            text='← Back', command=previous_page, font=('Microsoft JhengHei UI', 13))
        back_to_register.place(x=0, y=0)
        forgot_button = Button(text='Forgot Password', command=forgot_account, font=(
            'Microsoft JhengHei UI', 13), width=15)
        forgot_button.pack(pady=10)
        master.resizable(False, False)
        master.protocol('WM_DELETE_WINDOW', False)
        master.mainloop()

    def leave_program():
        quit()
    window = Tk()
    center_window(window)
    label1 = Label(window, text='Welcome to my game', font=24)
    label1.pack(padx=10, pady=10)
    label2 = Label(
        window, text='Would you like to login or register?', font=24)
    label2.pack(padx=10, pady=40)
    register_button = Button(text='Register', font=(
        'Microsoft JhengHei UI', 13), width=10, command=registration_page)
    register_button.place(x=200, y=150)
    login_button = Button(text='Login', font=(
        'Microsoft JhengHei UI', 13), width=10, command=login_page)
    login_button.place(x=400, y=150)
    leave = Button(text='Quit', font=(
        'Microsoft JhengHei UI', 13), command=leave_program)
    leave.place(x=652, y=468)
    window.resizable(False, False)
    window.protocol('WM_DELETE_WINDOW', False)
    window.mainloop()


# ---------------------------------------------------------------------------------- (Register)
op_lst = ['+', '-', '/', 'x']
lst_length = 10
points = 0
lives = 4
difficulty = 1


def valid_input_only(string):
    try:
        if round(float(string), 1) == 0.0:
            return True
        elif float(string) or int(string):
            return True
    except Exception:
        return False


def previous_game_page():
    global lst_length, points, lives, difficulty
    lst_length = 10
    points = 0
    lives = 4
    difficulty = 1
    game_difficulty_choice()


def get_answer(num1, rand_op, num2):
    if rand_op == '+':
        correct_ans = num1 + num2
    elif rand_op == '-':
        correct_ans = num1 - num2
    elif rand_op == 'x':
        correct_ans = num1 * num2
    else:
        correct_ans = round((num1 / num2), 1)
    return correct_ans


def create_rand_questions(difficulty):
    def generate_rand_nums(difficulty):
        max_num = '1' + '0' * difficulty
        num1, num2 = random.randint(
            1, int(max_num)), random.randint(1, int(max_num))
        rand_op = random.choice(op_lst)
        return num1, rand_op, num2
    num_lst = []
    counter = 0
    if difficulty in [2, 3]:
        counter = difficulty * 2
    lst_length = len(num_lst)
    while counter != 0:
        num1, rand_op, num2 = generate_rand_nums(difficulty)
        results = get_answer(num1, rand_op, num2)
        if isinstance(results, float):
            float_ans = (num1, rand_op, num2)
            num_lst.append(float_ans)
            lst_length += 1
            counter -= 1
    while lst_length != 10:
        num1, rand_op, num2 = generate_rand_nums(difficulty)
        results = get_answer(num1, rand_op, num2)
        if isinstance(results, int):
            int_ans = (num1, rand_op, num2)
            num_lst.append(int_ans)
            lst_length += 1
    return num_lst


def clear_animation(box_name, message):
    box_name['text'] = f'{message}'
    box_name['fg'] = 'black'


def remove_point_box_animation(point_box, output_box, button, window_name=Tk):
    window_name.after(1000, clear_animation, point_box, f'Points: {points}')
    window_name.after(1000, clear_box, output_box, button)


def remove_lives_box_animation(lives_box, output_box, button, window_name=Tk):
    window_name.after(1000, clear_animation, lives_box, f'Lives: {lives}')
    window_name.after(1000, clear_box, output_box, button)


def choose_rand_questions(num_lst=list):
    random_tuple = random.choice(num_lst)
    num_lst.remove(random_tuple)
    num1, rand_op, num2 = random_tuple[0], random_tuple[1], random_tuple[2]
    return num1, rand_op, num2


def game_difficulty_choice():
    game = Tk()
    center_window(game)
    label = Label(text='Select Game Difficulty', font=('Comic Sans MS', 14))
    label.pack()

    def easy():
        game.destroy()
        easy_difficulty()

    def medium():
        game.destroy()
        medium_difficulty()

    def hard():
        game.destroy()
        hard_difficulty()
    easy_button = Button(text='Easy', font=(
        'Microsoft JhengHei UI', 13), width=10, command=easy)
    easy_button.pack(pady=10)
    medium_button = Button(text='Medium', font=(
        'Microsoft JhengHei UI', 13), width=10, command=medium)
    medium_button.pack(pady=10)
    hard_button = Button(text='Hard', font=(
        'Microsoft JhengHei UI', 13), width=10, command=hard)
    hard_button.pack(pady=10)
    game.resizable(False, False)
    game.protocol('WM_DELETE_WINDOW', False)
    game.mainloop()


def easy_difficulty():
    global lst_length, points, lives, difficulty
    easy_game = Tk()
    center_window(easy_game)
    num_lst = create_rand_questions(difficulty)
    num1, rand_op, num2 = choose_rand_questions(num_lst)
    lst_length -= 1

    def answer_checker():
        global lst_length, points, lives, difficulty
        nonlocal num1, rand_op, num2, num_lst
        user_ans = answer_box.get()
        if user_ans.count('-') == 1:
            negative_answer = True
            user_ans = user_ans[1:]
        else:
            negative_answer = False
        if valid_input_only(user_ans):
            user_ans = round(float(user_ans), 1)
            if negative_answer:
                user_ans = '-' + str(user_ans)
                user_ans = float(user_ans)
            correct_ans = round(float(get_answer(num1, rand_op, num2)), 1)
            if user_ans == correct_ans:
                button['state'] = 'disabled'
                answer_box.delete(0, END)
                points += 1
                output_box['text'] = 'Correct!'
                output_box['fg'] = 'green'
                point_box['text'] = '+1'
                point_box['fg'] = 'green'
                remove_point_box_animation(
                    point_box, output_box, button, easy_game)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0, END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                remove_lives_box_animation(
                    lives_box, output_box, button, easy_game)
            if lst_length == 0:
                lst_length = 10
                num_lst = create_rand_questions(difficulty)
                num1, rand_op, num2 = choose_rand_questions(num_lst)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1, rand_op, num2 = choose_rand_questions(num_lst)
                lst_length -= 1
            if lives == 0:
                easy_game.destroy()
                game_over_screen()
        else:
            button['state'] = 'disabled'
            output_box['text'] = 'Invalid Input!'
            output_box['fg'] = 'red'
            easy_game.after(1000, clear_box, output_box, button)
            answer_box.delete(0, END)
        question_box['text'] = f'What is {num1} {rand_op} {num2}?'

    def quit_game():
        nonlocal num_lst
        num_lst.clear()
        easy_game.destroy()
        game_over_screen()

    def previous_page():
        nonlocal num_lst
        num_lst.clear()
        easy_game.destroy()
        previous_game_page()
    label = Label(text='Easy Difficulty', font=('Helvatical Bold', 15))
    label.pack()
    question_box = Message(text=f'What is {num1} {rand_op} {num2}?', font=(
        'Comic Sans MS', 17), width=350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='', font=24, width=350)
    output_box.pack()
    button = Button(text='Enter', font=(
        'Microsoft JhengHei UI', 13), command=answer_checker)
    button.pack()
    point_box = Message(text=f'Points: {points}', font=(
        'Comic Sans MS', 14), width=350)
    point_box.place(x=250, y=230)
    lives_box = Message(text=f'Lives: {lives}', font=(
        'Comic Sans MS', 14), width=350)
    lives_box.place(x=350, y=230)
    quit_button = Button(text='Quit Game', font=(
        'Microsoft JhengHei UI', 13), command=quit_game)
    quit_button.place(x=600, y=468)
    back_to_game_choice = Button(
        text='← Back', command=previous_page, font=('Microsoft JhengHei UI', 13))
    back_to_game_choice.place(x=0, y=0)
    easy_game.resizable(False, False)
    easy_game.protocol('WM_DELETE_WINDOW', False)
    easy_game.mainloop()


def medium_difficulty():
    global lst_length, points, lives, difficulty
    lives -= 1
    difficulty += 1
    medium_game = Tk()
    center_window(medium_game)
    num_lst = create_rand_questions(difficulty)
    num1, rand_op, num2 = choose_rand_questions(num_lst)
    lst_length -= 1
    result = get_answer(num1, rand_op, num2)

    def answer_checker():
        global lst_length, points, lives, difficulty
        nonlocal num1, rand_op, num2, num_lst
        user_ans = answer_box.get()
        if user_ans.count('-') == 1:
            negative_answer = True
            user_ans = user_ans[1:]
        else:
            negative_answer = False
        if valid_input_only(user_ans):
            user_ans = round(float(user_ans), 1)
            if negative_answer:
                user_ans = '-' + str(user_ans)
                user_ans = float(user_ans)
            correct_ans = round(float(get_answer(num1, rand_op, num2)), 1)
            if user_ans == correct_ans:
                button['state'] = 'disabled'
                answer_box.delete(0, END)
                points += 1
                output_box['text'] = 'Correct!'
                output_box['fg'] = 'green'
                point_box['text'] = '+1'
                point_box['fg'] = 'green'
                remove_point_box_animation(
                    point_box, output_box, button, medium_game)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0, END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                remove_lives_box_animation(
                    lives_box, output_box, button, medium_game)
            if lst_length == 0:
                lst_length = 10
                num_lst = create_rand_questions(difficulty)
                num1, rand_op, num2 = choose_rand_questions(num_lst)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1, rand_op, num2 = choose_rand_questions(num_lst)
                lst_length -= 1
            next_question_answer = get_answer(num1, rand_op, num2)
            if lives == 0:
                medium_game.destroy()
                game_over_screen()
            if isinstance(next_question_answer, float):
                question_box['text'] = f'What is {num1} {rand_op} {num2} to 1 d.p?'
            else:
                question_box['text'] = f'What is {num1} {rand_op} {num2}?'
        else:
            button['state'] = 'disabled'
            output_box['text'] = 'Invalid Input!'
            output_box['fg'] = 'red'
            medium_game.after(1000, clear_box, output_box, button)
            answer_box.delete(0, END)

    def quit_game():
        nonlocal num_lst
        num_lst.clear()
        medium_game.destroy()
        game_over_screen()

    def previous_page():
        nonlocal num_lst
        num_lst.clear()
        medium_game.destroy()
        previous_game_page()
    label = Label(text='Medium Difficulty', font=('Helvatical Bold', 15))
    label.pack()
    if isinstance(result, float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text=f'{question_msg}', font=(
        'Comic Sans MS', 17), width=350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='', font=24, width=350)
    output_box.pack()
    button = Button(text='Enter', font=(
        'Microsoft JhengHei UI', 13), command=answer_checker)
    button.pack()
    point_box = Message(text=f'Points: {points}', font=(
        'Comic Sans MS', 14), width=350)
    point_box.place(x=250, y=230)
    lives_box = Message(text=f'Lives: {lives}', font=(
        'Comic Sans MS', 14), width=350)
    lives_box.place(x=350, y=230)
    quit_button = Button(text='Quit Game', font=(
        'Microsoft JhengHei UI', 13), command=quit_game)
    quit_button.place(x=600, y=468)
    back_to_game_choice = Button(
        text='← Back', command=previous_page, font=('Microsoft JhengHei UI', 13))
    back_to_game_choice.place(x=0, y=0)
    medium_game.resizable(False, False)
    medium_game.protocol('WM_DELETE_WINDOW', False)
    medium_game.mainloop()


def hard_difficulty():
    global lst_length, points, lives, difficulty
    lives -= 2
    difficulty += 2
    hard_game = Tk()
    center_window(hard_game)
    num_lst = create_rand_questions(difficulty)
    num1, rand_op, num2 = choose_rand_questions(num_lst)
    lst_length -= 1
    result = get_answer(num1, rand_op, num2)

    def answer_checker():
        global lst_length, points, lives, difficulty
        nonlocal num1, rand_op, num2, num_lst
        user_ans = answer_box.get()
        if user_ans.count('-') == 1:
            negative_answer = True
            user_ans = user_ans[1:]
        else:
            negative_answer = False
        if valid_input_only(user_ans):
            user_ans = round(float(user_ans), 1)
            if negative_answer:
                user_ans = '-' + str(user_ans)
                user_ans = float(user_ans)
            correct_ans = round(float(get_answer(num1, rand_op, num2)), 1)
            if user_ans == correct_ans:
                button['state'] = 'disabled'
                answer_box.delete(0, END)
                points += 1
                output_box['text'] = 'Correct!'
                output_box['fg'] = 'green'
                point_box['text'] = '+1'
                point_box['fg'] = 'green'
                remove_point_box_animation(
                    point_box, output_box, button, hard_game)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0, END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                remove_lives_box_animation(
                    lives_box, output_box, button, hard_game)
            if lst_length == 0:
                lst_length = 10
                num_lst = create_rand_questions(difficulty)
                num1, rand_op, num2 = choose_rand_questions(num_lst)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1, rand_op, num2 = choose_rand_questions(num_lst)
                lst_length -= 1
            next_question_answer = get_answer(num1, rand_op, num2)
            if lives == 0:
                hard_game.destroy()
                game_over_screen()
            if isinstance(next_question_answer, float):
                question_box['text'] = f'What is {num1} {rand_op} {num2} to 1 d.p?'
            else:
                question_box['text'] = f'What is {num1} {rand_op} {num2}?'
        else:
            button['state'] = 'disabled'
            output_box['text'] = 'Invalid Input!'
            output_box['fg'] = 'red'
            hard_game.after(1000, clear_box, output_box, button)
            answer_box.delete(0, END)

    def quit_game():
        nonlocal num_lst
        num_lst.clear()
        hard_game.destroy()
        game_over_screen()

    def previous_page():
        nonlocal num_lst
        num_lst.clear()
        hard_game.destroy()
        previous_game_page()
    label = Label(text='Hard Difficulty', font=('Helvatical Bold', 15))
    label.pack()
    if isinstance(result, float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text=f'{question_msg}', font=(
        'Comic Sans MS', 17), width=350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='', font=24, width=350)
    output_box.pack()
    button = Button(text='Enter', font=(
        'Microsoft JhengHei UI', 13), command=answer_checker)
    button.pack()
    point_box = Message(text=f'Points: {points}', font=(
        'Comic Sans MS', 14), width=350)
    point_box.place(x=250, y=230)
    lives_box = Message(text=f'Lives: {lives}', font=(
        'Comic Sans MS', 14), width=350)
    lives_box.place(x=350, y=230)
    quit_button = Button(text='Quit Game', font=(
        'Microsoft JhengHei UI', 13), command=quit_game)
    quit_button.place(x=600, y=468)
    back_to_game_choice = Button(
        text='← Back', command=previous_page, font=('Microsoft JhengHei UI', 13))
    back_to_game_choice.place(x=0, y=0)
    hard_game.resizable(False, False)
    hard_game.protocol('WM_DELETE_WINDOW', False)
    hard_game.mainloop()


def game_over_screen():
    global points, difficulty, holding_username
    difficulty_lst = ['Easy', 'Medium', 'Hard']
    user_personal_best = []
    game_over = Tk()
    center_window(game_over)
    cursor.execute(
        f'SELECT best_point , best_points_difficulty FROM user_info WHERE username = "{holding_username}"')
    for i in cursor:
        user_personal_best.append(i)
    if user_personal_best[0][0] < points:
        cursor.execute(
            f'UPDATE user_info SET best_point = {points} , best_points_difficulty = "{difficulty_lst[difficulty-1]}" WHERE username = "{holding_username}"')
        db.commit()
    label = Label(text='GAME OVER!', fg='red', width=50, font=('System', 24))
    label.pack()
    label2 = Label(
        text=f'You scored {points} in {difficulty_lst[difficulty-1]} difficulty', font=('Comic Sans MS', 14), width=350)
    label2.pack(pady=20)

    def retry_button():
        global lives, points, difficulty
        lives = 4
        points = 0
        difficulty = 1
        game_over.destroy()
        game_difficulty_choice()

    def lb_button():
        game_over.destroy()
        display_lb()

    def quit_button():
        game_over.destroy()
    retry = Button(text='Play again', font=(
        'Microsoft JhengHei UI', 13), width=15, command=retry_button)
    retry.pack(pady=10)
    lb = Button(text='See Leaderboards', font=(
        'Microsoft JhengHei UI', 13), width=15, command=lb_button)
    lb.pack(pady=10)
    quit = Button(text='Quit', font=('Microsoft JhengHei UI', 13),
                  width=15, command=quit_button)
    quit.pack(pady=10)
    game_over.resizable(False, False)
    game_over.protocol('WM_DELETE_WINDOW', False)
    game_over.mainloop()


def display_lb():
    global holding_username
    position = 0

    def leave_program():
        quit()

    def shift_lb():
        nonlocal position
        position += 1
        if position == 3:
            position = 0
        if position == 0:
            first_place['text'] = f'🏆🥇 {hard_lb[0][0]} scored {hard_lb[0][1]} in {hard_lb[0][2]} difficulty'
            second_place['text'] = f'🏆🥈 {hard_lb[1][0]} scored {hard_lb[1][1]} in {hard_lb[1][2]} difficulty'
            third_place['text'] = f'🏆🥉 {hard_lb[2][0]} scored {hard_lb[2][1]} in {hard_lb[2][2]} difficulty'
        elif position == 1:
            first_place['text'] = f'🏆🥇 {medium_lb[0][0]} scored {medium_lb[0][1]} in {medium_lb[0][2]} difficulty'
            second_place['text'] = f'🏆🥈 {medium_lb[1][0]} scored {medium_lb[1][1]} in {medium_lb[1][2]} difficulty'
            third_place['text'] = f'🏆🥉 {medium_lb[2][0]} scored {medium_lb[2][1]} in {medium_lb[2][2]} difficulty'
        elif position == 2:
            first_place['text'] = f'🏆🥇 {easy_lb[0][0]} scored {easy_lb[0][1]} in {easy_lb[0][2]} difficulty'
            second_place['text'] = f'🏆🥈 {easy_lb[1][0]} scored {easy_lb[1][1]} in {easy_lb[1][2]} difficulty'
            third_place['text'] = f'🏆🥉 {easy_lb[2][0]} scored {easy_lb[2][1]} in {easy_lb[2][2]} difficulty'
    hard_lb = []
    medium_lb = []
    easy_lb = []
    user_points = []
    leaderboard = Tk()
    center_window(leaderboard)
    label = Label(text='Leaderboard', font=24)
    label.pack()
    cursor.execute(
        'SELECT username , best_point , best_points_difficulty FROM user_info WHERE best_points_difficulty = "Hard" ORDER BY best_point DESC LIMIT 3;')
    for i in cursor:
        hard_lb.append(i)
    cursor.execute(
        'SELECT username , best_point , best_points_difficulty FROM user_info WHERE best_points_difficulty = "Medium" ORDER BY best_point DESC LIMIT 3;')
    for i in cursor:
        medium_lb.append(i)
    cursor.execute(
        'SELECT username , best_point , best_points_difficulty FROM user_info WHERE best_points_difficulty = "Easy" ORDER BY best_point DESC LIMIT 3;')
    for i in cursor:
        easy_lb.append(i)
    cursor.execute(
        f'SELECT best_point , best_points_difficulty FROM user_info WHERE username = "{holding_username}"')
    for i in cursor:
        user_points.append(i)
    first_place = Label(
        text=f'🏆🥇 {hard_lb[0][0]} scored {hard_lb[0][1]} in {hard_lb[0][2]} difficulty', font=('Comic Sans MS', 14))
    first_place.pack(pady=10)
    second_place = Label(
        text=f'🏆🥈 {hard_lb[1][0]} scored {hard_lb[1][1]} in {hard_lb[1][2]} difficulty', font=('Comic Sans MS', 14))
    second_place.pack(pady=10)
    third_place = Label(
        text=f'🏆🥉 {hard_lb[2][0]} scored {hard_lb[2][1]} in {hard_lb[2][2]} difficulty', font=('Comic Sans MS', 14))
    third_place.pack(pady=10)
    button = Button(text='Next', font=(
        'Microsoft JhengHei UI', 13), command=shift_lb)
    button.pack(pady=10)
    if user_points[0][1] is None:
        msg = f'No personal best record'
    else:
        msg = f'Personal best is {user_points[0][0]} in {user_points[0][1]} difficulty'
    user_best = Label(text=f'{msg}', font=24)
    user_best.pack(pady=10)
    leave = Button(text='Quit', font=(
        'Microsoft JhengHei UI', 13), command=leave_program)
    leave.place(x=652, y=468)
    leaderboard.resizable(False, False)
    leaderboard.protocol('WM_DELETE_WINDOW', False)
    leaderboard.mainloop()


# ----------------------------------------------------------------------------------(Game System)

def main_game():
    authenticate_user()
    game_difficulty_choice()


main_game()
