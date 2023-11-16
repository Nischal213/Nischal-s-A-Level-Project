from tkinter import *
import random
change = False
question_number = 0
op_lst = ['+','-','/','x']
lst_length = 10
points = 0
lives = 4
difficulty = 1
def clear_box(box_type , button_name):
            box_type['text'] = ''
            button_name['state'] = 'normal'
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
def game_difficulty_choice():
    game = Tk()
    game.geometry('700x500')
    label = Label(text = 'Select Game Difficulty' , font = 24)
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
    easy_button = Button(text = 'Easy' , font = 24 , width = 10 , command = easy)
    easy_button.pack(pady=10)
    medium_button = Button(text = 'Medium' , font = 24 , width = 10 , command = medium)
    medium_button.pack(pady=10)
    hard_button = Button(text = 'Hard' , font = 24 , width = 10 , command = hard)
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
                easy_game.after(1000, clear_animation , point_box , f'Points: {points}')
                easy_game.after(1000 , clear_box , output_box , button)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                easy_game.after(1000, clear_animation , lives_box , f'Lives: {lives}')
                easy_game.after(1000 , clear_box , output_box , button)
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
        easy_game.destroy()
        game_over_screen()
    label = Label(text = 'Easy Difficulty', font =  ('Helvatical Bold' , 15))
    label.pack()
    question_box = Message(text = f'What is {num1} {rand_op} {num2}?' , font =  ('Helvatical Bold' , 15) , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='' , font = 24 , width = 350)
    output_box.pack()
    button = Button(text = 'Enter', font = ('Microsoft JhengHei UI',13) , command = checker)
    button.pack()
    point_box = Message(text = f'Points: {points}' , font =  ('Malgun Gothic',14)  , width = 350)
    point_box.place(x = 250 , y = 210)
    lives_box = Message(text = f'Lives: {lives}' , font =  ('Malgun Gothic',14) , width = 350)
    lives_box.place(x = 350 , y = 209)
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
                medium_game.after(1000, clear_animation , point_box , f'Points: {points}')
                medium_game.after(1000 , clear_box , output_box , button)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                medium_game.after(1000, clear_animation , lives_box , f'Lives: {lives}')
                medium_game.after(1000 , clear_box , output_box , button)
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
        medium_game.destroy()
        game_over_screen()
    label = Label(text = 'Meidum Difficulty', font = ('Helvatical Bold' , 15))
    label.pack()
    if isinstance(result,float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text = f'{question_msg}' , font =  ('Helvatical Bold' , 15) , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='' , font = 24 , width = 350)
    output_box.pack()
    button = Button(text = 'Enter', font = 24 , command = checker)
    button.pack()
    point_box = Message(text = f'Points: {points}' , font = ('Malgun Gothic',14) , width = 350)
    point_box.place(x = 250 , y = 200)
    lives_box = Message(text = f'Lives: {lives}' , font = ('Malgun Gothic',14) , width = 350)
    lives_box.place(x = 350 , y = 199)
    quit_button = Button(text = 'Quit Game' , font = ('Microsoft JhengHei UI',13) , command = quit_game)
    quit_button.place(x = 611, y = 468)
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
                hard_game.after(1000, clear_animation , point_box , f'Points: {points}')
                hard_game.after(1000 , clear_box , output_box , button)
            else:
                button['state'] = 'disabled'
                answer_box.delete(0,END)
                lives -= 1
                output_box['text'] = 'Incorrect!'
                output_box['fg'] = 'red'
                lives_box['text'] = '-1'
                lives_box['fg'] = 'red'
                hard_game.after(1000, clear_animation , lives_box , f'Lives: {lives}')
                hard_game.after(1000 , clear_box , output_box , button)
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
        hard_game.destroy()
        game_over_screen()
    label = Label(text = 'Hard Difficulty', font = ('Helvatical Bold' , 15))
    label.pack()
    if isinstance(result,float):
        question_msg = f'What is {num1} {rand_op} {num2} to 1 d.p?'
    else:
        question_msg = f'What is {num1} {rand_op} {num2}?'
    question_box = Message(text = f'{question_msg}' , font = ('Helvatical Bold' , 15) , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    output_box = Message(text='' , font = 24 , width = 350)
    output_box.pack()
    button = Button(text = 'Enter', font = 24 , command = checker)
    button.pack()
    point_box = Message(text = f'Points: {points}' , font = ('Malgun Gothic',14) , width = 350)
    point_box.place(x = 250 , y = 200)
    lives_box = Message(text = f'Lives: {lives}' , font = ('Malgun Gothic',14) , width = 350)
    lives_box.place(x = 350 , y = 199)
    quit_button = Button(text = 'Quit Game' , font = ('Microsoft JhengHei UI',13) , command = quit_game)
    quit_button.place(x = 611, y = 468)
    hard_game.mainloop()


def game_over_screen():
    global points , difficulty
    difficulty_lst = ['Easy','Medium','Hard']
    game_over = Tk()
    game_over.geometry('700x500')
    label = Label(text = 'GAME OVER!' , fg = 'red' , width = 50 , font =  ('Helvatical Bold' , 24))
    label.pack()
    label2 = Label(text = f'You scored {points} in {difficulty_lst[difficulty-1]} difficulty' , font = 24 , width = 350)
    label2.pack(pady=20)
    def retry_button():
        game_over.destroy()
        game_difficulty_choice()
    def lb_button():
        game_over.destroy()
        display_lb()
    def quit_button():
        game_over.destroy()
    retry = Button(text = 'Play again' , font = 24 , width = 15 , command = retry_button)
    retry.pack(pady=10)
    lb = Button(text = 'See Leaderboards' , font = 24 , width = 15 , command = lb_button)
    lb.pack(pady=10)
    quit = Button(text = 'Quit', font = 24 , width = 15 , command = quit_button)
    quit.pack(pady=10)
    game_over.mainloop()

def display_lb():
    print('To be worked on')

easy_game_UI()
