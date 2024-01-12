import random
import string
import pymysql
from tkinter import *

#Allows access to the database by making a cursor
db = pymysql.connect(
    host='localhost',
    user='root',
    password='root',
    database='game_db')
cursor = db.cursor()

#global variables
attempt = 1
holding_username = '' #Variable that holds the user's username
window_destroyed = False

#A function that gives the illusion of an animation
#This is done by changing the text to blank after a set number of seconds
def clear_box(box_name, button_name):
    box_name['text'] = ''
    button_name['state'] = 'normal'

#A function that customizes any window given the name of it
def customize_window(window_name=Tk):
    window_name.title("Nischal's A-level project") #Sets title of window
    window_name.resizable(False, False) #Disables expanding the window
    window_name.protocol('WM_DELETE_WINDOW', False) #Disables closing the window
    window_name['bg'] = 'light blue' #Sets bg color
    screen_width = int((window_name.winfo_screenwidth()) / 2 - 375) #Specifies screen width by taking screen width of user's device into consideration
    screen_height = int((window_name.winfo_screenheight()) / 2 - 250) #Same as above but for screen height
    window_name.geometry(f'700x500+{screen_width}+{screen_height}') #Specifies the position where the window is supposed to be placed

#All of registration/log in is handled by this function
def authenticate_user():

    def registration_page():
        global window_destroyed
        window.destroy()
        window_destroyed = True #Variable value changed to indicate the main window has been destroyed
        root = Tk()
        customize_window(root)
        change = False #Change is a variable that is true if one of the requirements has been violated
        question_number = 0 #allows the user to proceed to the next question by changing the question index

        def change_questions():
            global hold_first_name, hold_password, hold_last_name
            nonlocal change, question_number
            if question_number == 0:
                first_name = place_holder_box.get()
                last_name = place_holder_box2.get()
                #From here
                if first_name == '' or last_name == '':
                    change = True
                    error_string = 'Name cannot be blank'
                elif not (first_name.isalpha()) or not (last_name.isalpha()):
                    change = True
                    error_string = 'Name cannot contain special chars'
                if change:
                    button['state'] = 'disabled'
                    error_message['text'] = error_string
                    root.after(2000, clear_box, error_message, button)
                    change = False
                #To here
                #The code snippet validates the last name and first name
                #By checking if they are letters and dont have numbers
                else:
                    question_number += 1
                    #place holders so that the first name and last name can be updated on the db later
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
                    #From here
                    if not (password == conf_password):
                        error_string = 'Passwords not matching'
                        change = True
                    if password == '' or conf_password == '':
                        error_string = 'Password cannot be blank'
                        change = True
                    #To here
                    #The code snippet validates the password by making sure they match and aren't empty
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
                        #The button is disabled then enabled again to prevent python from immediately getting
                        #The user's input as in the code above I have emptied both message boxes so
                        #python would get the empty inputs and display an error message because of my validation
                        button['state'] = 'disabled'
                        root.after(100, clear_box, error_message, button)
            if question_number == 2:
                #function to check if a sting has any special characters
                def has_special_chars(str):
                    for i in str:
                        if i in string.punctuation:
                            return True
                    else:
                        return False

                #function to check if an email string is valid
                def email_validator(str):
                    if has_special_chars(str):
                        #checks if the email string contains at least 1 '.' and only 1 '@'
                        if str.count('.') != 0 and str.count('@') == 1:
                            #checks if there is anything before the '@' symbol
                            before_at_symbol = len(str[:str.index('@')])
                            #checks if there is anything after the '@'symbol
                            after_at_symbol = len(str[str.index('@') + 1:])
                            if before_at_symbol > 0 and after_at_symbol > 0:
                                return True
                            else:
                                return False
                        else:
                            return False
                    else:
                        return False
                #function similar to clear_box() but this one takes no parameter and affects 2 boxes
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
                    #Using the cursor to check if anyone else has the same username
                    cursor.execute(
                        f'SELECT username FROM user_info WHERE username = "{username}"')
                    #cursor.rowcount function allows us to check if any rows were affected when searching for the username
                    #If a row was affected then the rowcount would be greater than 0 indicating that someone already has the username
                    username_row = cursor.rowcount
                    #Using the cursor to check if anyone else has the same email
                    cursor.execute(
                        f'SELECT email FROM user_info WHERE email = "{email}" ')
                    #If a row was affected then the rowcount would be greater than 0 indicating that someone already has the email
                    email_row = cursor.rowcount
                    #From here
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
                    #To here
                    #The code snippet validates the username and email
                    #The username is validated by checking if rowcount = 0, it is within 3-20 chars long,
                    #It has no special_chars and is not solely consisting of digits
                    #The email is validated by checking if rowcount = 0 and a function email_validator()
                    if change:
                        username_error_box['text'] = username_error_string
                        email_error_box['text'] = email_error_string
                        button['state'] = 'disabled'
                        root.after(2000, new_clear_box)
                        change = False
                    else:
                        global holding_username
                        #global variable is changed
                        holding_username = username
                        button['state'] = 'disabled'
                        #updating the db with the new values of first name, last name,username, password and a set value of best_points
                        cursor.execute(
                            f'INSERT INTO user_info (first_name,last_name,username,pass_word,email,best_point) VALUES ("{hold_first_name}","{hold_last_name}","{username}","{hold_password}","{email}",0)')
                        db.commit()
                        root.destroy()
                        direct_to_login()

        def previous_page():
            global window_destroyed
            window_destroyed = False
            root.destroy()
            authenticate_user()
        label = Label(text='Register Page', font=('Comic Sans MS' , 16) , bg = 'light blue')
        label.pack()
        place_holder_label = Label(text='First name', font=('Comic Sans MS' , 16) , bg = 'light blue')
        place_holder_label.pack(pady=10)
        place_holder_box = Entry()
        place_holder_box.pack()
        place_holder_label2 = Label(text='Last name', font=('Comic Sans MS' , 16) , bg = 'light blue')
        place_holder_label2.pack(pady=10)
        place_holder_box2 = Entry()
        place_holder_box2.pack()
        button = Button(text='Enter', command=change_questions,
                        font=('Constantia', 13))
        button.pack(pady=10)
        error_message = Message(text='', fg='red', font=('Constantia', 13), width=450 , bg = 'light blue')
        error_message.pack()
        username_error_box = Message(text='', font=('Constantia', 13), fg='red', width=550 , bg = 'light blue')
        username_error_box.place(x=420, y=84)
        email_error_box = Message(text='', font=('Constantia', 13), fg='red', width=550 , bg = 'light blue')
        email_error_box.place(x=420, y=158)
        back_to_register = Button(
            text='← Back', command=previous_page, font=(
                'Constantia', 13) , fg = 'blue')
        back_to_register.place(x=0, y=0)
        root.mainloop()

    def direct_to_login():
        real = Tk()
        customize_window(real)

        def to_login_page():
            real.destroy()
            login_page()
        label = Label(text='An account has been created!', font=('Comic Sans MS' , 16) , bg = 'light blue')
        label.pack()
        button = Button(
            text='Go to login page',
            command=to_login_page,
            font=(
                'Constantia',
                13),
            width=20)
        button.pack(pady=20)
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
            def lockdown(box_type , button_name , button_name2 , button_name3):
                box_type['text'] = ''
                button_name['state'] , button_name2['state'] , button_name3['state'] = 'normal' , 'normal' , 'normal'
            cursor.execute(
                f'SELECT username, pass_word FROM user_info WHERE username = "{username}" AND pass_word = "{password}"')
            if cursor.rowcount == 0:
                error_string = f'Invalid password or username! Attempt: {attempt}'
                pass_flag = False
            if not (pass_flag) and error_string:
                if attempt == 6:
                    attempt = 1
                    error_message['text'] = 'Too many failed attempts in a row, Temporarily on lockdown for 30 seconds'
                    check_button['state'] = 'disabled'
                    back_to_register['state'] = 'disabled'
                    forgot_button['state'] = 'disabled'
                    master.after(30000, lockdown , error_message, check_button , back_to_register , forgot_button)
                else:
                    check_button['state'] = 'disabled'
                    error_message['text'] = error_string
                    master.after(2000, clear_box, error_message, check_button)
                    attempt += 1
            else:
                global holding_username
                holding_username = username
                master.destroy()

        def forgot_account():
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
                    contact = Tk()

                    def leave_program():
                        quit()
                    customize_window(contact)
                    label = Label(text='Contact Page', font=('Comic Sans MS' , 16) , bg = 'light blue')
                    label.pack()
                    support = Label(
                        text='Contact us here at 07855609342', font=('Comic Sans MS' , 16) , bg = 'light blue')
                    support.pack(pady=20)
                    leave = Button(text='Quit', font=(
                        'Constantia', 13), command=leave_program , width = 7 , fg = 'red')
                    leave.place(x=621, y=468)
                    contact.mainloop()

            def previous_page():
                global window_destroyed
                window_destroyed = True
                new_master.destroy()
                login_page()
            new_master = Tk()
            customize_window(new_master)
            label = Label(text='Forgot Page', font=('Comic Sans MS' , 16), bg = 'light blue')
            label.pack()
            username_label = Label(text='Username', font=('Comic Sans MS' , 16) , bg = 'light blue')
            username_label.pack(pady=10)
            username_entry = Entry()
            username_entry.pack()
            username_box = Message(text='', font=('Constantia' , 13), fg='red', width=350 , bg = 'light blue')
            username_box.pack()
            email_label = Label(text='Email', font=('Comic Sans MS' , 16)  , bg = 'light blue')
            email_label.pack()
            email_entry = Entry()
            email_entry.pack()
            email_box = Message(text='', font=('Constantia' , 13), fg='red', width=350 , bg = 'light blue')
            email_box.pack()
            button = Button(
                text='Enter',
                font=(
                    'Constantia',
                    13),
                command=forgot_account_verification)
            button.pack(pady=10)
            back_to_login = Button(
                text='← Back', command=previous_page, font=(
                    'Microsoft JhengHei UI', 13) , fg = 'blue')
            back_to_login.place(x=0, y=0)
            new_master.mainloop()

        def previous_page():
            global window_destroyed
            window_destroyed = False
            master.destroy()
            authenticate_user()
        master = Tk()
        customize_window(master)
        label = Label(text='Login Page', font=('Comic Sans MS' , 16) , bg = 'light blue')
        label.pack()
        username_label = Label(text='Username:', font=('Comic Sans MS' , 16) , bg = 'light blue')
        username_label.pack(pady=10)
        username_entry = Entry()
        username_entry.pack()
        password_label = Label(text='Password:', font=('Comic Sans MS' , 16) , bg = 'light blue')
        password_label.pack(pady=10)
        password_entry = Entry()
        password_entry.pack()
        check_button = Button(
            text='Enter', command=account_verification, font=(
                'Constantia', 13), width=10)
        check_button.pack(pady=10)
        error_message = Message(text='', font=('Constantia' , 13), fg='red', width=550 , bg = 'light blue')
        error_message.pack()
        back_to_register = Button(
            text='← Back', command=previous_page, font=(
                'Constantia', 13) , fg = 'blue')
        back_to_register.place(x=0, y=0)
        forgot_button = Button(
            text='Forgot Password',
            command=forgot_account,
            font=(
                'Constantia',
                13),
            width=15)
        forgot_button.pack(pady=10)
        master.mainloop()

    def leave_program():
        quit()
    window = Tk()
    customize_window(window)
    label1 = Label(text='Welcome to my game', font=('Comic Sans MS' , 16) , bg = 'light blue')
    label1.pack(padx=10, pady=10)
    label2 = Label(
        text='Would you like to login or register?', font=('Comic Sans MS' , 16) , bg = 'light blue')
    label2.pack(padx=10, pady=40)
    register_button = Button(text='Register', font=(
        'Constantia', 13), width=10, command=registration_page)
    register_button.place(x=200, y=150)
    login_button = Button(text='Login', font=(
        'Constantia', 13), width=10, command=login_page)
    login_button.place(x=400, y=150)
    leave = Button(text='Quit', font=(
        'Constantia', 13), width=7, command=leave_program , fg = 'red')
    leave.place(x=621, y=468)
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


def get_answer(num1, rand_op, num2, algebra=False, num3=None):
    if not (algebra):
        if rand_op == '+':
            correct_ans = num1 + num2
        elif rand_op == '-':
            correct_ans = num1 - num2
        elif rand_op == 'x':
            correct_ans = num1 * num2
        else:
            correct_ans = round((num1 / num2), 1)
        return correct_ans
    else:
        index = op_lst.index(rand_op)
        if index % 2 == 0:
            index += 1
        else:
            index -= 1
        if index == 3:
            changed_op = '*'
        else:
            changed_op = op_lst[index]
        correct_ans = eval(f'( {num3} {changed_op} {num2} ) / {num1}')
        if eval(f'{num3} {changed_op} {num2}') % num1 == 0:
            return int(correct_ans)
        else:
            return round(correct_ans, 1)


def create_rand_questions(difficulty):
    def generate_rand_nums(store_difficulty):
        if store_difficulty == 3:
            store_difficulty = 1
        max_num = '1' + '0' * store_difficulty
        if difficulty != 3:
            num1, num2 = random.randint(
                1, int(max_num)), random.randint(1, int(max_num))
            rand_op = random.choice(op_lst)
            return num1, rand_op, num2
        else:
            num1, num2, num3 = random.randint(
                1, int(max_num)), random.randint(
                1, int(max_num)), random.randint(
                1, int(max_num))
            rand_op = random.choice(op_lst)
            return num1, rand_op, num2, num3
    num_lst = []
    counter = 0
    store_difficulty = difficulty
    algebra = False
    if difficulty == 3:
        algebra = True
    if difficulty == 2:
        counter = difficulty * 2
    lst_length = len(num_lst)
    while counter != 0:
        num1, rand_op, num2 = generate_rand_nums(store_difficulty)
        results = get_answer(num1, rand_op, num2, algebra)
        if isinstance(results, float):
            float_ans = (num1, rand_op, num2)
            num_lst.append(float_ans)
            lst_length += 1
            counter -= 1
    if algebra:
        for i in range(10):
            num1, rand_op, num2, num3 = generate_rand_nums(store_difficulty)
            ans = get_answer(num1, rand_op, num2, algebra, num3)
            question = (num1, rand_op, num2, num3, ans)
            num_lst.append(question)
            lst_length += 1
    while lst_length != 10:
        num1, rand_op, num2 = generate_rand_nums(difficulty)
        results = get_answer(num1, rand_op, num2, algebra)
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


def choose_rand_questions(num_lst=list, algebra=False):
    if not (algebra):
        random_tuple = random.choice(num_lst)
        num_lst.remove(random_tuple)
        num1, rand_op, num2 = random_tuple[0], random_tuple[1], random_tuple[2]
        return num1, rand_op, num2
    else:
        random_tuple = random.choice(num_lst)
        num_lst.remove(random_tuple)
        num1, rand_op, num2, num3, ans = random_tuple[0], random_tuple[
            1], random_tuple[2], random_tuple[3], random_tuple[4]
        return num1, rand_op, num2, num3, ans


def game_difficulty_choice():
    game = Tk()
    customize_window(game)
    label = Label(text='Select Game Difficulty', font=('Comic Sans MS', 14) , bg = 'light blue')
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
        'Constantia', 13), width=10, fg='green', command=easy)
    easy_button.pack(pady=10)
    medium_button = Button(text='Medium', font=(
        'Constantia', 13), width=10, fg='orange', command=medium)
    medium_button.pack(pady=10)
    hard_button = Button(text='Hard', font=(
        'Constantia', 13), width=10, fg='red', command=hard)
    hard_button.pack(pady=10)
    game.mainloop()


def easy_difficulty():
    global lst_length, points, lives, difficulty
    easy_game = Tk()
    customize_window(easy_game)
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
            if back_to_game_choice:
                back_to_game_choice.destroy()
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
    label = Label(text='Easy Difficulty', font=('Comic Sans MS' , 16) , bg = 'light blue')
    label.pack()
    question_box = Message(text=f'What is {num1} {rand_op} {num2}?', font=(
        'Comic Sans MS', 16), width=350 , bg = 'light blue')
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='', font=('Constantia' , 13), width=350 , bg = 'light blue')
    output_box.pack()
    button = Button(text='Enter', font=(
        'Constantia', 13), command=answer_checker)
    button.pack()
    point_box = Message(text=f'Points: {points}', font=(
        'Comic Sans MS', 14), width=350 , bg = 'light blue')
    point_box.place(x=250, y=230)
    lives_box = Message(text=f'Lives: {lives}', font=(
        'Comic Sans MS', 14), width=350 , bg = 'light blue')
    lives_box.place(x=350, y=230)
    quit_button = Button(text='Quit', font=(
        'Constantia', 13), command=quit_game , width = 7 , fg = 'red')
    quit_button.place(x=621, y=468)
    back_to_game_choice = Button(
        text='← Back', command=previous_page, font=(
            'Constantia', 13) , fg = 'blue')
    back_to_game_choice.place(x=0, y=0)
    easy_game.mainloop()


def medium_difficulty():
    global lst_length, points, lives, difficulty
    lives -= 1
    difficulty += 1
    medium_game = Tk()
    customize_window(medium_game)
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
            if back_to_game_choice:
                back_to_game_choice.destroy()
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
    label = Label(text='Medium Difficulty', font=('Comic Sans MS' , 16) , bg = 'light blue')
    label.pack()
    if isinstance(result, float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text=f'{question_msg}', font=(
        'Comic Sans MS', 16), width=350 , bg = 'light blue')
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='', font=('Constantia' , 13), width=350 , bg = 'light blue')
    output_box.pack()
    button = Button(text='Enter', font=(
        'Constantia', 13), command=answer_checker)
    button.pack()
    point_box = Message(text=f'Points: {points}', font=(
        'Comic Sans MS', 14), width=350 , bg = 'light blue')
    point_box.place(x=250, y=230)
    lives_box = Message(text=f'Lives: {lives}', font=(
        'Comic Sans MS', 14), width=350 , bg = 'light blue')
    lives_box.place(x=350, y=230)
    quit_button = Button(text='Quit', font=(
        'Constantia', 13), command=quit_game , width = 7 , fg = 'red')
    quit_button.place(x=621, y=468)
    back_to_game_choice = Button(
        text='← Back', command=previous_page, font=(
            'Constantia', 13) , fg = 'blue')
    back_to_game_choice.place(x=0, y=0)
    medium_game.mainloop()


def hard_difficulty():
    global lst_length, points, lives, difficulty
    lives -= 2
    difficulty += 2
    hard_game = Tk()
    customize_window(hard_game)
    num_lst = create_rand_questions(difficulty)
    num1, rand_op, num2, num3, ans = choose_rand_questions(num_lst, True)
    lst_length -= 1

    def answer_checker():
        global lst_length, points, lives, difficulty
        nonlocal num1, rand_op, num2, num3, ans, num_lst
        user_ans = answer_box.get()
        if valid_input_only(user_ans):
            user_ans = round(float(user_ans), 1)
            if user_ans == float(ans):
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
            if back_to_game_choice:
                back_to_game_choice.destroy()
            if lst_length == 0:
                lst_length = 10
                num_lst = create_rand_questions(difficulty)
                num1, rand_op, num2, num3, ans = choose_rand_questions(
                    num_lst, True)
                lst_length -= 1
            elif lives != 0 and lst_length != 0:
                num1, rand_op, num2, num3, ans = choose_rand_questions(
                    num_lst, True)
                lst_length -= 1
            next_question_answer = ans
            if lives == 0:
                hard_game.destroy()
                game_over_screen()
            if isinstance(next_question_answer, float):
                question_box['text'] = f'Find y to 1 d.p: {num1}y {rand_op} {num2} = {num3}'
            else:
                question_box['text'] = f'Find y: {num1}y {rand_op} {num2} = {num3}'
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
    label = Label(text='Hard Difficulty', font=('Comic Sans MS' , 16) , bg = 'light blue')
    label.pack()
    if isinstance(ans, float):
        question_msg = f'Find y to 1 d.p: {num1}y {rand_op} {num2} = {num3}'
    else:
        question_msg = f'Find y: {num1}y {rand_op} {num2} = {num3}'
    question_box = Message(text=f'{question_msg}', font=(
        'Comic Sans MS', 16), width=350 , bg = 'light blue')
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='', font=('Constantia' , 13), width=350 , bg = 'light blue')
    output_box.pack()
    button = Button(text='Enter', font=(
        'Constantia', 13), command=answer_checker)
    button.pack()
    point_box = Message(text=f'Points: {points}', font=(
        'Comic Sans MS', 14), width=350 , bg = 'light blue')
    point_box.place(x=250, y=230)
    lives_box = Message(text=f'Lives: {lives}', font=(
        'Comic Sans MS', 14), width=350 , bg = 'light blue')
    lives_box.place(x=350, y=230)
    quit_button = Button(text='Quit', font=(
        'Constantia', 13), command=quit_game , width = 7 , fg = 'red')
    quit_button.place(x=621, y=468)
    back_to_game_choice = Button(
        text='← Back', command=previous_page, font=(
            'Constantia', 13) , fg = 'blue')
    back_to_game_choice.place(x=0, y=0)
    hard_game.mainloop()


def game_over_screen():
    global points, difficulty, holding_username
    difficulty_lst = ['Easy', 'Medium', 'Hard']
    user_personal_best = []
    game_over = Tk()
    customize_window(game_over)
    cursor.execute(
        f'SELECT best_point , best_points_difficulty FROM user_info WHERE username = "{holding_username}"')
    for i in cursor:
        user_personal_best.append(i)
    if user_personal_best[0][0] < points:
        cursor.execute(
            f'UPDATE user_info SET best_point = {points} , best_points_difficulty = "{difficulty_lst[difficulty-1]}" WHERE username = "{holding_username}"')
        db.commit()
    label = Label(text='GAME OVER!', fg='red', width=50, font=('System', 24) , bg = 'light blue')
    label.pack()
    label2 = Label(
        text=f'You scored {points} in {difficulty_lst[difficulty-1]} difficulty',
        font=(
            'Comic Sans MS',
            14),
        width=350 , bg = 'light blue')
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
        'Constantia', 13), width=15, command=retry_button , fg = 'green')
    retry.pack(pady=10)
    lb = Button(text='See Leaderboards', font=(
        'Constantia', 13), width=15, command=lb_button , fg = 'teal')
    lb.pack(pady=10)
    quit = Button(text='Quit', font=('Constantia', 13),
                  width=15, command=quit_button , fg = 'red')
    quit.pack(pady=10)
    game_over.mainloop()


def display_lb():
    global holding_username
    def leave_program():
        quit()

    def shift_lb_easy():
        first_place['text'] = f'🏆🥇 {easy_lb[0][0]} scored {easy_lb[0][1]} in {easy_lb[0][2]} difficulty'
        second_place['text'] = f'🏆🥈 {easy_lb[1][0]} scored {easy_lb[1][1]} in {easy_lb[1][2]} difficulty'
        third_place['text'] = f'🏆🥉 {easy_lb[2][0]} scored {easy_lb[2][1]} in {easy_lb[2][2]} difficulty'

    def shift_lb_medium():
        first_place['text'] = f'🏆🥇 {medium_lb[0][0]} scored {medium_lb[0][1]} in {medium_lb[0][2]} difficulty'
        second_place['text'] = f'🏆🥈 {medium_lb[1][0]} scored {medium_lb[1][1]} in {medium_lb[1][2]} difficulty'
        third_place['text'] = f'🏆🥉 {medium_lb[2][0]} scored {medium_lb[2][1]} in {medium_lb[2][2]} difficulty'

    def shift_lb_hard():
        first_place['text'] = f'🏆🥇 {hard_lb[0][0]} scored {hard_lb[0][1]} in {hard_lb[0][2]} difficulty'
        second_place['text'] = f'🏆🥈 {hard_lb[1][0]} scored {hard_lb[1][1]} in {hard_lb[1][2]} difficulty'
        third_place['text'] = f'🏆🥉 {hard_lb[2][0]} scored {hard_lb[2][1]} in {hard_lb[2][2]} difficulty'
    hard_lb = []
    medium_lb = []
    easy_lb = []
    user_points = []
    leaderboard = Tk()
    customize_window(leaderboard)
    label = Label(text='Leaderboard', font=('Comic Sans MS' , 16) , bg = 'light blue')
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
        text=f'🏆🥇 {hard_lb[0][0]} scored {hard_lb[0][1]} in {hard_lb[0][2]} difficulty',
        font=(
            'Comic Sans MS',
            14) , bg = 'light blue' , fg = 'orange')
    first_place.pack(pady=10)
    second_place = Label(
        text=f'🏆🥈 {hard_lb[1][0]} scored {hard_lb[1][1]} in {hard_lb[1][2]} difficulty',
        font=(
            'Comic Sans MS',
            14) , bg = 'light blue', fg = 'gray')
    second_place.pack(pady=10)
    third_place = Label(
        text=f'🏆🥉 {hard_lb[2][0]} scored {hard_lb[2][1]} in {hard_lb[2][2]} difficulty',
        font=(
            'Comic Sans MS',
            14) , bg = 'light blue', fg = 'brown')
    third_place.pack(pady=10)
    display_medium = Button(text='Medium', font=(
        'Constantia', 13), fg = 'orange' , command=shift_lb_medium)
    display_medium.place(x=300, y=195)
    display_hard = Button(text='Hard', font=(
        'Constantia', 13), fg = 'red' , command=shift_lb_hard)
    display_hard.place(x=225, y=195)
    display_easy = Button(text='Easy', font=(
        'Constantia', 13), fg = 'green' , command=shift_lb_easy)
    display_easy.place(x=400, y=195)
    if user_points[0][1] is None:
        msg = f'No personal best record'
    else:
        msg = f'Personal best is {user_points[0][0]} in {user_points[0][1]} difficulty'
    user_best = Label(text=f'{msg}', font=('Comic Sans MS' , 16) , bg = 'light blue')
    user_best.pack(pady=60)
    leave = Button(text='Quit', font=(
        'Constantia', 13), command=leave_program , width = 7 , fg = 'red')
    leave.place(x=621, y=468)
    leaderboard.mainloop()


# ----------------------------------------------------------------------------------(Game System)

def main_game():
    authenticate_user()
    game_difficulty_choice()

   
authenticate_user()
