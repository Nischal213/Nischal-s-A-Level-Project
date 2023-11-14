# Nischal-s-A-Level-Project
from tkinter import *
import random
change = False
question_number = 0
op_lst = ['+','-','/','x']
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
            correct_ans = num1 / num2
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

def main_game():
    easy_mode , medium_mode , hard_mode = False , False ,False
    game = Tk()
    game.geometry('700x500')
    label = Label(text = 'Select Game Difficulty' , font = 24)
    label.pack()
    def easy():
        game.destroy()
        easy_game_UI()
    easy_button = Button(text = 'Easy' , font =24 , width = 10 , command = easy)
    easy_button.pack(pady=10)
    game.mainloop()

def easy_game_UI():
    def display_question(num_lst , lst_length):
        random_tuple = random.choice(num_lst)
        lst_length -= 1 
        num_lst.remove(random_tuple)
        num1 , rand_op , num2 = random_tuple[0] , random_tuple[1] , random_tuple[2]
        return num1 , rand_op , num2
    easy_game = Tk()
    easy_game.geometry('700x500')
    lst_length = 10
    num_lst = create_questions(1)
    num1 , rand_op , num2 = display_question(num_lst , lst_length)
    def checker():
        global points , lives , num1 , rand_op , num2
        user_ans = answer_box.get()
        correct_ans = giveAnswer(num1 , rand_op , num2)
        if user_ans == str(correct_ans) :
            button['state'] = 'disabled'
            points = points + 1
            question_box['text'] = 'Correct!'
            question_box['fg'] = 'green'
            easy_game.after(2000 , clear_box , question_box , button)
        else:
            button['state'] = 'disabled'
            points = points + 1
            question_box['text'] = 'Incorrect!'
            question_box['fg'] = 'red'
            easy_game.after(2000 , clear_box , question_box , button)
        if lst_length == 0:
            num1 , rand_op , num2 = display_question(num_lst , lst_length)
        elif lives != 0 and lst_length != 0:
            num_lst = create_questions(1)
            num1 , rand_op , num2 = display_question(num_lst , lst_length)
        else:
            exit()
        question_box['text'] = f'What is {num1} {rand_op} {num2}?'
    label = Label(text = 'Easy Difficulty', font = 24)
    label.pack()
    question_box = Message(text = f'What is {num1} {rand_op} {num2}?' , font = 24 , width = 350)
    question_box.pack(pady=20)
    answer_box = Entry()
    answer_box.pack(pady=10)
    button = Button(text = 'Enter', font = 24 , command = checker)
    button.pack(pady=10)
    easy_game.mainloop()

main_game()
