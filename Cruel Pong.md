from tkinter import *
import os  # Optional for sounds on macOS
import time  # For delays

# Create game window
win = Tk()
win.title("Pong Game Python 3 Tkinter")
win.resizable(0, 0)
canvas = Canvas(win, width=800, height=600, bg='black')
canvas.pack()

# Initialize scores
p1score = 0
p2score = 0

# Create paddles and ball
p1 = canvas.create_rectangle(40, 255, 60, 345, fill='white')
p2 = canvas.create_rectangle(740, 255, 760, 345, fill='white')
ball = canvas.create_oval(400, 300, 420, 320, fill='white')
score = canvas.create_text(380, 50, fill='red', font="Courier 24 normal", text=f"Player 1: {p1score}   Player 2: {p2score}")

# Ball movement speed
x, y = 4, 4

# Paddle movement direction
p1_move_up = False
p1_move_down = False

# Paddle 1 movement
def move_p1(event):
    global p1_move_up, p1_move_down
    if event.keysym == "w":
        p1_move_up = True
    elif event.keysym == "s":
        p1_move_down = True

def stop_p1(event):
    global p1_move_up, p1_move_down
    if event.keysym == "w":
        p1_move_up = False
    elif event.keysym == "s":
        p1_move_down = False

def update_p1_position():
    p1_coords = canvas.coords(p1)
    if p1_move_up and p1_coords[1] > 0:
        canvas.move(p1, 0, -10)
    elif p1_move_down and p1_coords[3] < 600:
        canvas.move(p1, 0, 10)
    win.after(20, update_p1_position)

# AI for Paddle 2
def ai_move_p2():
    p2_coords = canvas.coords(p2)
    ball_coords = canvas.coords(ball)
    ball_center_y = (ball_coords[1] + ball_coords[3]) / 2
    p2_center_y = (p2_coords[1] + p2_coords[3]) / 2

    # Move AI paddle gradually towards ball's center
    if ball_center_y > p2_center_y + 5:
        canvas.move(p2, 0, 3)  # Smooth movement down
    elif ball_center_y < p2_center_y - 5:
        canvas.move(p2, 0, -3)  # Smooth movement up

# Assign keys for Player 1
canvas.bind_all("<KeyPress-w>", move_p1)
canvas.bind_all("<KeyPress-s>", move_p1)
canvas.bind_all("<KeyRelease-w>", stop_p1)
canvas.bind_all("<KeyRelease-s>", stop_p1)

# Game loop
try:
    update_p1_position()  # Start updating player 1's position
    while True:
        # Move the ball
        canvas.move(ball, x, y)
        ball_coords = canvas.coords(ball)
        p1_coords = canvas.coords(p1)
        p2_coords = canvas.coords(p2)

        # Check for scoring conditions
        if ball_coords[0] <= 0:  # Ball passed left side
            x = -x
            canvas.coords(ball, 400, 300, 420, 320)
            p2score += 1
            canvas.delete(score)
            score = canvas.create_text(380, 50, fill='red', font="Courier 24 normal",
                                       text=f"Player 1: {p1score}   Player 2: {p2score}")
            os.system('say "score"') if os.name == 'posix' else None

        if ball_coords[2] >= 800:  # Ball passed right side
            x = -x
            canvas.coords(ball, 400, 300, 420, 320)
            p1score += 1
            canvas.delete(score)
            score = canvas.create_text(380, 50, fill='red', font="Courier 24 normal",
                                       text=f"Player 1: {p1score}   Player 2: {p2score}")
            os.system('say "score"') if os.name == 'posix' else None

        # Ball bouncing on top or bottom edges
        if ball_coords[1] <= 0 or ball_coords[3] >= 600:
            y = -y

        # Ball bouncing on paddles
        if (740 <= ball_coords[2] <= 760 and ball_coords[3] >= p2_coords[1] and ball_coords[1] <= p2_coords[3] and x > 0):
            x = -x
        if (40 <= ball_coords[0] <= 60 and ball_coords[3] >= p1_coords[1] and ball_coords[1] <= p1_coords[3] and x < 0):
            x = -x

        # Move AI paddle (Player 2)
        ai_move_p2()

        win.update()
        time.sleep(0.017)  # Add delay for consistent game speed
except TclError:
    # Exit gracefully when the window is closed
    print("Game exited.")
