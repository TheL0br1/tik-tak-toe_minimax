import random
import time
import tkinter as tk
from tkinter import messagebox


moves = 0

def isMovesLeft(board):
    for i in range(7):
        for j in range(7):
            if (board[i][j] == ' '):
                return True
    return False

def evaluate(b):
    for i in range(7):
        for j in range(7 - 3):
            if b[i][j:j+4] == 'OOOO':
                return 10
            if b[i][j:j+4] == 'XXXX':
                return -10
            if b[j][i] == b[j+1][i] == b[j+2][i] == b[j+3][i] == 'O':
                return 10
            if b[j][i] == b[j+1][i] == b[j+2][i] == b[j+3][i] == 'X':
                return -10
    for i in range(7 - 3):
        for j in range(7 - 3):
            if b[i][j] == b[i+1][j+1] == b[i+2][j+2] == b[i+3][j+3] == 'O':
                return 10
            if b[i][j] == b[i+1][j+1] == b[i+2][j+2] == b[i+3][j+3] == 'X':
                return -10
            if b[i][j+3] == b[i+1][j+2] == b[i+2][j+1] == b[i+3][j] == 'O':
                return 10
            if b[i][j+3] == b[i+1][j+2] == b[i+2][j+1] == b[i+3][j] == 'X':
                return -10
    return 0
def minimax(board, depth, alpha, beta, isMax):
    score = evaluate(board)

    if depth > (2+int(moves/7)) or abs(score) == 10:
        return score
    if not isMovesLeft(board):
        return 0

    if isMax:
        best = -1000
        for i in range(7):
            for j in range(7):
                if board[i][j] == ' ':
                    board[i][j] = 'O'
                    best = max(best, minimax(board, depth + 1, alpha, beta, not isMax))
                    alpha = max(alpha, best)
                    board[i][j] = ' '
                    if beta <= alpha:
                        break
        return best
    else:
        best = 1000
        for i in range(7):
            for j in range(7):
                if board[i][j] == ' ':
                    board[i][j] = 'X'
                    best = min(best, minimax(board, depth + 1, alpha, beta, not isMax))
                    beta = min(beta, best)
                    board[i][j] = ' '
                    if beta <= alpha:
                        break
        return best

def findBestMove(board):
    bestVal = -1000
    bestMove = (-1, -1)
    alpha = -1000
    beta = 1000

    for i in range(7):
        for j in range(7):
            if board[i][j] == ' ':
                board[i][j] = 'O'
                moveVal = minimax(board, 0, alpha, beta, False)
                board[i][j] = ' '

                if moveVal > bestVal:
                    bestMove = (i, j)
                    bestVal = moveVal

                alpha = max(alpha, bestVal)
    return bestMove

class Player:
    def __init__(self, symbol):
        self.symbol = symbol
        self.score = 0


class RandomBot:
    def make_move(self, board):
        available_moves = [(i, j) for i in range(len(board)) for j in range(len(board[i])) if board[i][j] == ' ']
        return random.choice(available_moves) if available_moves else None


class TicTacToe:

    def __init__(self, size=7):
        self.size = size
        self.board = [[' ' for _ in range(size)] for _ in range(size)]
        self.players = []
        self.winner_board = [[' ' for _ in range(size)] for _ in range(size)]

    def copy_2d_array(self, original):
        rows = len(original)
        cols = len(original[0])

        # Using nested loops to copy the array
        copied = [[original[i][j] for j in range(cols)] for i in range(rows)]

        return copied

    def add_player(self, player):
        if len(self.players) < 2:
            self.players.append(player)
        else:
            print("Максимальна кількість гравців досягнута.")

    def print_board(self):
        for row in self.board:
            print('|'.join(row))
            print('-' * (self.size * 2 - 1))

    def make_move(self, player, row, col):
        if self.is_valid_move(row, col):
            self.board[row][col] = player.symbol
            self.winner_board[row][col] = player.symbol
            return True
        else:
            print("Недійсний хід. Спробуйте ще раз.")
            return False

    def is_valid_move(self, row, col):
        return 0 <= row < self.size \
               and 0 <= col < self.size \
               and self.board[row][col] == ' '

    def check_winner(self, player):
        # Перевірка горизонталей і вертикалей
        for i in range(self.size):
            for j in range(self.size - 3):
                if all(self.winner_board[i][j + k] == player.symbol for k in range(4)):
                    for k in range(4):
                        self.winner_board[i][j + k] = '_'

                    return True
                if all(self.winner_board[j + k][i] == player.symbol for k in range(4)):
                    for k in range(4):
                        self.winner_board[j + k][i] = '_'

                    return True

        # Перевірка діагоналей
        for i in range(self.size - 3):
            for j in range(self.size - 3):
                if all(self.winner_board[i + k][j + k] == player.symbol for k in range(4)):
                    for k in range(4):
                        self.winner_board[i + k][j + k] = '_'

                    return True
                if all(self.winner_board[i + k][j + 3 - k] == player.symbol for k in range(4)):
                    for k in range(4):
                        self.winner_board[i + k][j + 3 - k] = '_'
                    return True

        return False

    def play_game(self):
        current_player = 0

        while any(' ' in row for row in self.board):
            self.print_board()
            print(f"Гравець {self.players[current_player].symbol}, ваш хід:")

            row = int(input("Введіть номер рядка: "))
            col = int(input("Введіть номер стовпця: "))

            if self.make_move(self.players[current_player], row, col):
                if self.check_winner(self.players[current_player]):
                    print(f"Гравець {self.players[current_player].symbol} виграв!")
                    break

                current_player = (current_player + 1) % 2

        print("Гра завершена.")


class TicTacToeGUI(TicTacToe):
    def __init__(self, master, players, size=7):
        super().__init__(size)
        self.players = players
        self.master = master
        self.score_labels = [tk.Label(self.master, text=f"Гравець {self.players[i].symbol}: {self.players[i].score}",
                                      font=('normal', 16), width=12, height=3)
                             for i in range(len(self.players))]
        self.buttons = [[None for _ in range(size)] for _ in range(size)]
        self.create_gui()

    def update_score_labels(self):
        for i, label in enumerate(self.score_labels):
            label.config(text=f"Гравець {self.players[i].symbol}: {self.players[i].score}")
            label.grid(row=self.size + 1, column=i * 4, columnspan=5, rowspan=5)

    def add_player_gui(self, player):
        self.score_labels.append(
            tk.Label(self.master, text=f"Гравець {player.symbol}: {player.score}", font=('normal', 20), width=3,
                     height=1))
        self.add_player(player)

    def create_gui(self):
        for i in range(self.size):
            for j in range(self.size):
                self.buttons[i][j] = tk.Button(self.master, text='', font=('normal', 20), width=3, height=1,
                                               command=lambda row=i, col=j: self.make_gui_move(row, col))
                self.buttons[i][j].grid(row=i, column=j)
        for i, label in enumerate(self.score_labels):
            label.grid(row=self.size + 1, column=i * 4, columnspan=5, rowspan=5)

    def make_gui_move(self, row, col):
        if self.make_move(self.players[0], row, col):
            self.buttons[row][col].config(text=self.players[0].symbol, state=tk.DISABLED)

            if self.check_winner(self.players[0]):
               # messagebox.showinfo("Гра закінчена", f"Гравець {self.players[0].symbol} виграв!")
                self.players[0].score += 1
                self.update_score_labels()
            self.make_bot_move()
            if self.check_winner(self.players[1]):
               # messagebox.showinfo("Гра закінчена", f"Гравець {self.players[1].symbol} виграв!")
                self.players[1].score += 1
                self.update_score_labels()

    def reset_game(self):
        # Скидає гру до початкового стану
        self.board = [[' ' for _ in range(self.size)] for _ in range(self.size)]
        for i in range(self.size):
            for j in range(self.size):
                self.buttons[i][j].config(text='', state=tk.NORMAL)

    def make_bot_move(self):
        bot = RandomBot()
        start_time = time.time()

        move = findBestMove(self.winner_board)
        end_time = time.time()
        execution_time = end_time - start_time
        print(f"Час на хід: {execution_time} секунд")

        global moves
        moves+=1
        if move:
            self.make_move(self.players[1], move[0], move[1])
            self.buttons[move[0]][move[1]].config(text=self.players[1].symbol, state=tk.DISABLED)


def main():
    root = tk.Tk()
    root.title("Хрестики-ноліки 7x7")

    human_vs_human = True

    if human_vs_human:
        player1 = Player('X')
        player2 = Player('O')
        game = TicTacToeGUI(root, [player1, player2])
    # game.add_player_gui(player1)
    # game.add_player_gui(player2)
    else:
        player1 = Player('X')
        player2 = Player('O')
        bot_game = TicTacToeGUI(root, [player1, player2])
        # bot_game.add_player_gui(player1)
        # bot_game.add_player_gui(player2)

    root.mainloop()


if __name__ == "__main__":
    main()
