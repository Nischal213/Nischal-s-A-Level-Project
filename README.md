import random
import string
import time
import pymysql
from tkinter import *
db = pymysql.connect(
    host = 'localhost',
    user = 'root',
    password = 'root',
    database = 'game_db')
cursor = db.cursor()
change = False
#---------------------------------------------------------------------------------- (Allowing Access to db)
question_number = 0
attempt = 1
real = False
root = False
def clear_box(box_type , button_name):
        box_type['text'] = ''
        button_name['state'] = 'normal'
def login_system():
    def register():
        window.destroy()
        root = Tk()
        root.geometry('700x500')
        def next():
            global change , question_number , hold_first_name , hold_last_name , hold_password
            if question_number == 0:
                first_name = place_holder_box.get()
                last_name = place_holder_box2.get()
                if first_name == '' or last_name == '':
                    change = True
                    error_string = 'Cannot be blank'
                elif not(first_name.isalpha) or not(last_name.isalpha()):
                    change = True
                    error_string = 'Name cannot contain special chars'
                if change:
                    button['state'] = 'disabled'
                    error_message['text'] = error_string
                    root.after(2000 , clear_box , error_message , button )
                    change = False
                else:
                    question_number += 1
                    hold_first_name , hold_last_name = first_name , last_name
                    place_holder_box.delete(0,END)
                    place_holder_box2.delete(0,END)
                    button['state'] = 'disabled'
                    root.after(100 , clear_box , error_message , button )
            if question_number == 1:
                place_holder_label['text'] = 'Password'
                place_holder_label2['text'] = 'Confirm Password'
                if button['state'] == 'normal':
                    password = place_holder_box.get()
                    conf_password = place_holder_box2.get()
                    if not(password == conf_password):
                        error_string = 'Passwords not matching'
                        change = True
                    if password == '' or conf_password == '':
                        error_string = 'Password cannot be blank'
                        change = True
                    if change:
                        error_message['text'] = error_string
                        button['state'] = 'disabled'
                        root.after(2000, clear_box , error_message , button)
                        change = False
                    else:
                        question_number += 1
                        hold_password = password
                        place_holder_box.delete(0,END)
                        place_holder_box2.delete(0,END)
                        button['state'] = 'disabled'
                        root.after(100 , clear_box , error_message , button )
            if question_number == 2:
                def has_special_chars(str):
                    for i in str:
                        if i in string.punctuation:
                            return True
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
                    cursor.execute(f'SELECT username FROM user_info WHERE username = "{username}"')
                    username_row = cursor.rowcount
                    cursor.execute(f'SELECT email FROM user_info WHERE email = "{email}" ')
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
                    if not(has_special_chars(email)) or email == '':
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
                        button['state'] = 'disabled'
                        cursor.execute(f'INSERT INTO user_info (first_name,last_name,username,pass_word,email,best_point) VALUES ("{hold_first_name}","{hold_last_name}","{username}","{hold_password}","{email}",0)')
                        db.commit()
                        root.destroy()
                        login()
        label = Label(text = 'Register Page' , font = 24)
        label.pack()
        place_holder_label = Label(text = 'First name:' , font = 24)
        place_holder_label.pack(pady=10)
        place_holder_box = Entry()
        place_holder_box.pack()
        place_holder_label2 = Label(text = 'Last name:', font = 24)
        place_holder_label2.pack(pady=10)
        place_holder_box2 = Entry()
        place_holder_box2.pack()
        button = Button(text = 'Enter' , command = next , font = 24)
        button.pack(pady=10)
        error_message = Message(text = '' , fg = 'red' , font = 24 , width = 450)
        error_message.pack()
        username_error_box = Message(text='' , font = 24 , fg = 'red' , width = 550)
        username_error_box.place(x = 420 , y = 64)
        email_error_box = Message(text='' , font = 24 , fg = 'red' , width = 550)
        email_error_box.place(x = 420 , y = 128)
        root.mainloop()
    def login():
            global real
            real = Tk()
            real.geometry('700x500')
            label = Label(real,text = 'An account has been created!' , font = 24)
            label.pack()
            button = Button(real, text = 'Go to login page' , command = login_page , font = 24 , width = 20)
            button.pack(pady=20)
            real.mainloop()
    def login_page():
            if real:
                real.destroy()
            if not(root):
                window.destroy()
            def checking():
                global attempt
                pass_flag = True
                error_string = ''
                username = username_entry.get()
                password = password_entry.get()
                cursor.execute(f'SELECT username, pass_word FROM user_info WHERE username = "{username}" AND pass_word = "{password}"')
                if cursor.rowcount == 0:
                    error_string = f'Invalid password or username!, Attempt: {attempt}'
                    pass_flag = False
                if not(pass_flag) and error_string:
                    if attempt == 6:
                        attempt = 0
                        error_message['text'] = 'Too many failed attempts in a row, Temporarily on lockdown for 30 seconds'
                        check_button['state'] = 'disabled'
                        master.after(30000, clear_box , error_message , check_button)
                    else:
                        check_button['state'] = 'disabled'
                        error_message['text'] = error_string
                        master.after(2000, clear_box , error_message , check_button)
                        attempt += 1
                else:
                    master.destroy()
            def forgot():
                master.destroy()
                def forgot_check():
                    def custom_clear_box():
                        username_box['text'] = ''
                        email_box['text'] = ''
                        button['state'] = 'normal'
                    modify = False
                    username = username_entry.get()
                    cursor.execute(f'SELECT username FROM user_info WHERE username = "{username}"')
                    if cursor.rowcount == 0:
                        username_box['text'] = 'Username does not exist'
                        modify = True
                    email = email_entry.get()
                    cursor.execute(f'SELECT email FROM user_info WHERE email = "{email}"')
                    if cursor.rowcount == 0:
                        email_box['text'] = 'Email does not exist'
                        modify = True
                    if modify:
                        button['state'] = 'disabled'
                        new_master.after(2000, custom_clear_box)
                    else:
                        new_master.destroy()
                new_master = Tk()
                new_master.geometry('700x500')
                label = Label(text = 'Forgot Page' , font = 24)
                label.pack()
                username = Label(text = 'Username' , font = 24)
                username.pack(pady=10)
                username_entry = Entry()
                username_entry.pack()
                username_box = Message(text='',font = 24 , fg = 'red' , width = 350)
                username_box.pack()
                email_label = Label(text = 'Email' , font = 24)
                email_label.pack()
                email_entry = Entry()
                email_entry.pack()
                email_box = Message(text = '' , font = 24 , fg = 'red' , width = 350)
                email_box.pack()
                button = Button(text = 'Enter' , font = 24,command = forgot_check)
                button.pack(pady=10)
                new_master.mainloop()
            master = Tk()
            master.geometry('700x500')
            label = Label(text = 'Login Page' , font = 24)
            label.pack()
            username = Label(text = 'Username:' , font = 24)
            username.pack(pady=10)
            username_entry = Entry()
            username_entry.pack()
            password = Label(master, text = 'Password:' , font = 24)
            password.pack(pady=10)
            password_entry = Entry()
            password_entry.pack()
            check_button = Button(text = 'Enter' , command = checking , font = 24 , width = 10)
            check_button.pack(pady=10)
            error_message = Message(text = '' , font = 24 , fg = 'red' , width = 550)
            error_message.pack()
            forgot_button = Button(text = 'Forgot Password' , command = forgot , font = 24 , width = 20)
            forgot_button.pack(pady=10)
            master.mainloop()
    window = Tk()
    window.geometry('700x500')
    label1 = Label(window, text = 'Welcome to my game' , font = 24)
    label1.pack(padx=10,pady=10)
    label2 = Label(window, text = 'Would you like to login or register?' , font = 24)
    label2.pack(padx=10,pady=40)
    register_button = Button(text = 'Register' , font = 24 , width = 10 , command=register)
    register_button.place(x = 200 , y = 150)
    login_button = Button(text = 'Login' , font = 24 , width = 10 , command=login_page)
    login_button.place(x = 400 , y = 150)
    window.mainloop()
#---------------------------------------------------------------------------------- (Register)
op_lst = ['+','-','/','x']
lst_length = 10
points = 0
lives = 4
difficulty = 1
def giveAnswer(num1,rand_op,num2):
        if rand_op == '+':
            correct_ans = num1 + num2
        elif rand_op == '-':
            correct_ans = num1 - num2
        elif rand_op == 'x':
            correct_ans = num1 * num2
        else:
            correct_ans = round((num1 / num2),1)
        return correct_ans
def generate_rand_nums(difficulty):
    max_num = '1' + '0' * difficulty
    num1 , num2 = random.randint(1,int(max_num)) , random.randint(1,int(max_num))
    rand_op = random.choice(op_lst)
    return num1 , rand_op , num2
def create_questions(difficulty):
    num_lst = []
    counter = 0
    if difficulty in [2,3]:
        counter = difficulty * 2
    lst_length = len(num_lst)
    while counter != 0:
        num1 , rand_op , num2 = generate_rand_nums(difficulty)
        results = giveAnswer(num1 , rand_op , num2)
        if isinstance(results,float):
            float_ans = (num1,rand_op,num2)
            num_lst.append(float_ans)
            lst_length += 1
            counter -= 1
    while lst_length != 10:
        num1 , rand_op , num2 = generate_rand_nums(difficulty)
        results = giveAnswer(num1 , rand_op , num2)
        if isinstance(results,int):
            int_ans = (num1,rand_op,num2)
            num_lst.append(int_ans)
            lst_length += 1
    return num_lst
def clear_animation(box_name , message):
    box_name['text'] = f'{message}'
    box_name['fg'] = 'black'
def remove_point_box_animation(window_name , point_box , output_box , button):
    window_name.after(1000, clear_animation , point_box , f'Points: {points}')
    window_name.after(1000 , clear_box , output_box , button)
def remove_lives_box_animation(window_name , lives_box , output_box , button):
    window_name.after(1000, clear_animation , lives_box , f'Lives: {lives}')
    window_name.after(1000 , clear_box , output_box , button)
def game_difficulty_choice():
    game = Tk()
    game.geometry('700x500')
    label = Label(text = 'Select Game Difficulty' , font = ('Comic Sans MS',14))
    label.pack()
    def easy():
        game.destroy()
        easy_game_UI()
    def medium():
        game.destroy()
        medium_game_UI()
    def hard():
        game.destroy()
        hard_game_UI()
    easy_button = Button(text = 'Easy' , font = ('Microsoft JhengHei UI',13) , width = 10 , command = easy)
    easy_button.pack(pady=10)
    medium_button = Button(text = 'Medium' , font = ('Microsoft JhengHei UI',13) , width = 10 , command = medium)
    medium_button.pack(pady=10)
    hard_button = Button(text = 'Hard' , font = ('Microsoft JhengHei UI',13) , width = 10 , command = hard)
    hard_button.pack(pady=10)
    game.mainloop()
def choose_question(num_lst=list):
    random_tuple = random.choice(num_lst)
    num_lst.remove(random_tuple)
    num1 , rand_op , num2 = random_tuple[0] , random_tuple[1] , random_tuple[2]
    return num1 , rand_op , num2
def easy_game_UI():
    global lst_length , points , lives , difficulty
    easy_game = Tk()
    easy_game.geometry('700x500')
    num_lst = create_questions(difficulty)
    num1 , rand_op , num2 = choose_question(num_lst)
    lst_length -= 1
    def checker():
        global lst_length , points , lives , difficulty
        nonlocal num1 , rand_op , num2 , num_lst
        user_ans = answer_box.get()
        if not(user_ans.isalpha()):
            correct_ans = giveAnswer(num1 , rand_op , num2)
            if user_ans == str(correct_ans) :
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                points += 1
                output_box['text'] = 'Correct!'
                output_box['fg'] = 'green'
                point_box['text'] = '+1'
                point_box['fg'] = 'green'
                remove_point_box_animation(easy_game , point_box , output_box , button)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                remove_lives_box_animation(easy_game , lives_box , output_box , button)
            if lst_length == 0:
                lst_length = 10
                num_lst = create_questions(difficulty)
                num1 , rand_op , num2 = choose_question(num_lst)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1 , rand_op , num2 = choose_question(num_lst)
                lst_length -= 1
            if lives == 0:
                easy_game.destroy()
                game_over_screen()
        else:
            button['state']= 'disabled'
            output_box['text'] = 'Invalid Input!'
            output_box['fg'] = 'red'
            easy_game.after(1000 , clear_box , output_box , button)
            answer_box.delete(0,END)
        question_box['text'] = f'What is {num1} {rand_op} {num2}?'
    def quit_game():
        nonlocal num_lst
        num_lst.clear()
        easy_game.destroy()
        game_over_screen()
    label = Label(text = 'Easy Difficulty', font =  ('Helvatical Bold' , 15))
    label.pack()
    question_box = Message(text = f'What is {num1} {rand_op} {num2}?' , font =  ('Comic Sans MS',17) , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='' , font = 24 , width = 350)
    output_box.pack()
    button = Button(text = 'Enter', font = ('Microsoft JhengHei UI',13) , command = checker)
    button.pack()
    point_box = Message(text = f'Points: {points}' , font =  ('Comic Sans MS',14)  , width = 350)
    point_box.place(x = 250 , y = 220)
    lives_box = Message(text = f'Lives: {lives}' , font =  ('Comic Sans MS',14) , width = 350)
    lives_box.place(x = 350 , y = 220)
    quit_button = Button(text = 'Quit Game' , font = ('Microsoft JhengHei UI',13) , command = quit_game)
    quit_button.place(x = 600, y = 468)
    easy_game.mainloop()
def medium_game_UI():
    global lst_length , points , lives , difficulty
    lives -= 1
    difficulty += 1
    medium_game = Tk()
    medium_game.geometry('700x500')
    num_lst = create_questions(difficulty)
    num1 , rand_op , num2 = choose_question(num_lst)
    lst_length -= 1
    result = giveAnswer(num1 , rand_op , num2)
    def checker():
        global lst_length , points , lives , difficulty
        nonlocal num1 , rand_op , num2 , num_lst
        user_ans = answer_box.get()
        if not(user_ans.isalpha()):
            correct_ans = giveAnswer(num1 , rand_op , num2)
            if user_ans == str(correct_ans) :
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                points += 1
                output_box['text'] = 'Correct!'
                output_box['fg'] = 'green'
                point_box['text'] = '+1'
                point_box['fg'] = 'green'
                remove_point_box_animation(medium_game , point_box , output_box , button)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                remove_lives_box_animation(medium_game , lives_box , output_box , button)
            if lst_length == 0:
                lst_length = 10
                num_lst = create_questions(difficulty)
                num1 , rand_op , num2 = choose_question(num_lst)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1 , rand_op , num2 = choose_question(num_lst)
                lst_length -= 1
            next_question_answer = giveAnswer(num1 , rand_op , num2)
            if lives == 0:
                medium_game.destroy()
                game_over_screen()
            if isinstance(next_question_answer,float):
                question_box['text'] = f'What is {num1} {rand_op} {num2} to 1 d.p?'
            else:
                question_box['text'] = f'What is {num1} {rand_op} {num2}?'
        else:
            button['state']= 'disabled'
            output_box['text'] = 'Invalid Input!'
            output_box['fg'] = 'red'
            medium_game.after(1000 , clear_box , output_box , button)
            answer_box.delete(0,END)
    def quit_game():
        nonlocal num_lst
        num_lst.clear()
        medium_game.destroy()
        game_over_screen()
    label = Label(text = 'Medium Difficulty', font = ('Helvatical Bold' , 15))
    label.pack()
    if isinstance(result,float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text = f'{question_msg}' , font =  ('Comic Sans MS',17) , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='' , font = 24 , width = 350)
    output_box.pack()
    button = Button(text = 'Enter', font = ('Microsoft JhengHei UI',13) , command = checker)
    button.pack()
    point_box = Message(text = f'Points: {points}' , font = ('Comic Sans MS',14) , width = 350)
    point_box.place(x = 250 , y = 220)
    lives_box = Message(text = f'Lives: {lives}' , font = ('Comic Sans MS',14) , width = 350)
    lives_box.place(x = 350 , y = 220)
    quit_button = Button(text = 'Quit Game' , font = ('Microsoft JhengHei UI',13) , command = quit_game)
    quit_button.place(x = 600, y = 468)
    medium_game.mainloop()
def hard_game_UI():
    global lst_length , points , lives , difficulty
    lives -= 2
    difficulty += 2
    hard_game = Tk()
    hard_game.geometry('700x500')
    num_lst = create_questions(difficulty)
    num1 , rand_op , num2 = choose_question(num_lst)
    lst_length -= 1
    result = giveAnswer(num1 , rand_op , num2)
    def checker():
        global lst_length , points , lives , difficulty
        nonlocal num1 , rand_op , num2 , num_lst
        user_ans = answer_box.get()
        if not(user_ans.isalpha()):
            correct_ans = giveAnswer(num1 , rand_op , num2)
            if user_ans == str(correct_ans) :
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                points += 1
                output_box['text'] = 'Correct!'
                output_box['fg'] = 'green'
                point_box['text'] = '+1'
                point_box['fg'] = 'green'
                remove_point_box_animation(hard_game , point_box , output_box , button)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                remove_lives_box_animation(hard_game , lives_box , output_box , button)
            if lst_length == 0:
                lst_length = 10
                num_lst = create_questions(difficulty)
                num1 , rand_op , num2 = choose_question(num_lst)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1 , rand_op , num2 = choose_question(num_lst)
                lst_length -= 1
            next_question_answer = giveAnswer(num1 , rand_op , num2)
            if lives == 0:
                hard_game.destroy()
                game_over_screen()
            if isinstance(next_question_answer,float):
                question_box['text'] = f'What is {num1} {rand_op} {num2} to 1 d.p?'
            else:
                question_box['text'] = f'What is {num1} {rand_op} {num2}?'
        else:
            button['state']= 'disabled'
            output_box['text'] = 'Invalid Input!'
            output_box['fg'] = 'red'
            hard_game.after(1000 , clear_box , output_box , button)
            answer_box.delete(0,END)
    def quit_game():
        nonlocal num_lst
        num_lst.clear()
        hard_game.destroy()
        game_over_screen()
    label = Label(text = 'Hard Difficulty', font = ('Helvatical Bold' , 15))
    label.pack()
    if isinstance(result,float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text = f'{question_msg}' , font = ('Comic Sans MS',17) , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='' , font = 24 , width = 350)
    output_box.pack()
    button = Button(text = 'Enter', font = ('Microsoft JhengHei UI',13) , command = checker)
    button.pack()
    point_box = Message(text = f'Points: {points}' , font = ('Comic Sans MS',14) , width = 350)
    point_box.place(x = 250 , y = 220)
    lives_box = Message(text = f'Lives: {lives}' , font = ('Comic Sans MS',14) , width = 350)
    lives_box.place(x = 350 , y = 220)
    quit_button = Button(text = 'Quit Game' , font = ('Microsoft JhengHei UI',13) , command = quit_game)
    quit_button.place(x = 600, y = 468)
    hard_game.mainloop()


def game_over_screen():
    global points , difficulty
    difficulty_lst = ['Easy','Medium','Hard']
    game_over = Tk()
    game_over.geometry('700x500')
    label = Label(text = 'GAME OVER!' , fg = 'red' , width = 50 , font =  ('System' , 24))
    label.pack()
    label2 = Label(text = f'You scored {points} in {difficulty_lst[difficulty-1]} difficulty' , font = ('Comic Sans MS',14) , width = 350)
    label2.pack(pady=20)
    def retry_button():
        global lives , points , difficulty
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
    retry = Button(text = 'Play again' , font = ('Microsoft JhengHei UI',13) , width = 15 , command = retry_button)
    retry.pack(pady=10)
    lb = Button(text = 'See Leaderboards' , font = ('Microsoft JhengHei UI',13) , width = 15 , command = lb_button)
    lb.pack(pady=10)
    quit = Button(text = 'Quit', font = ('Microsoft JhengHei UI',13) , width = 15 , command = quit_button)
    quit.pack(pady=10)
    game_over.mainloop()

def display_lb():
    print('To be worked on')


#----------------------------------------------------------------------------------(Game System)
game_over_screen()



#----------------------------------------------------------------------------------(Font Type)
import tkinter as tk
from tkinter import font
import pyperclip  # For copying to clipboard

root = tk.Tk()
root.title("Available Fonts in Tkinter")

# Get all available fonts
all_fonts = list(font.families())

# Initialize the font index for navigation
current_font_index = 0

# Function to update the displayed font
def update_font(delta):
    global current_font_index
    current_font_index = (current_font_index + delta) % len(all_fonts)
    selected_font = all_fonts[current_font_index]
    text_label.config(font=(selected_font, 20))  # Increased font size for better visibility
    text_label["text"] = f"Sample Text - Font: {selected_font}"
    root.clipboard_clear()
    root.clipboard_append(selected_font)

# Function to update the text label with the next font
def next_font():
    update_font(1)

# Function to update the text label with the previous font
def previous_font():
    update_font(-1)

# Function to copy the selected font to the clipboard
def copy_to_clipboard():
    selected_font = all_fonts[current_font_index]
    root.clipboard_clear()
    root.clipboard_append(selected_font)

# Create a tkinter Label to display text with different fonts
text_label = tk.Label(root, text="Sample Text", font=("Arial", 20))  # Default font for sample text
text_label.pack()

# Button to navigate to the previous font
prev_button = tk.Button(root, text="Previous", command=previous_font)
prev_button.pack(side=tk.LEFT, padx=5)

# Button to navigate to the next font
next_button = tk.Button(root, text="Next", command=next_font)
next_button.pack(side=tk.LEFT, padx=5)

# Button to copy the displayed font to clipboard
copy_button = tk.Button(root, text="Copy to Clipboard", command=copy_to_clipboard)
copy_button.pack(side=tk.RIGHT, padx=5)

# Display the initial font
text_label.config(font=(all_fonts[current_font_index], 20))
root.clipboard_clear()
root.clipboard_append(all_fonts[current_font_index])

root.mainloop()
