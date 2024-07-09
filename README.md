import pygame
import sys

# Constants
WIDTH, HEIGHT = 300, 400
LINE_WIDTH = 15
BOARD_ROWS, BOARD_COLS = 3, 3
SQUARE_SIZE = WIDTH // BOARD_COLS
CIRCLE_RADIUS = SQUARE_SIZE // 3
CIRCLE_WIDTH = 15
CROSS_WIDTH = 25
SPACE = SQUARE_SIZE // 4
RED = 'red'
BG_COLOR = 'black'
LINE_COLOR = 'white'
CIRCLE_COLOR = '#F5F5DC'
CROSS_COLOR = 'dark gray'
BUTTON_COLOR = '#343B3C'
BUTTON_TEXT_COLOR = '#D3D3D3'

pygame.init()

# Board class to manage the board state and drawing
class Board:
    def __init__(self):
        self.board = ['' for _ in range(9)]
        self.draw_lines()

    def draw_lines(self):
        screen.fill(BG_COLOR)
        # Horizontal lines
        pygame.draw.line(screen, LINE_COLOR, (0, SQUARE_SIZE), (WIDTH, SQUARE_SIZE), LINE_WIDTH)
        pygame.draw.line(screen, LINE_COLOR, (0, 2 * SQUARE_SIZE), (WIDTH, 2 * SQUARE_SIZE), LINE_WIDTH)
        # Vertical lines
        pygame.draw.line(screen, LINE_COLOR, (SQUARE_SIZE, 0), (SQUARE_SIZE, 3 * SQUARE_SIZE), LINE_WIDTH)
        pygame.draw.line(screen, LINE_COLOR, (2 * SQUARE_SIZE, 0), (2 * SQUARE_SIZE, 3 * SQUARE_SIZE), LINE_WIDTH)

    def draw_figures(self):
        for row in range(BOARD_ROWS):
            for col in range(BOARD_COLS):
                if self.board[row * BOARD_COLS + col] == 'X':
                    self.draw_cross(col, row)
                elif self.board[row * BOARD_COLS + col] == 'O':
                    self.draw_circle(col, row)

    def draw_circle(self, col, row):
        pygame.draw.circle(screen, CIRCLE_COLOR, (col * SQUARE_SIZE + SQUARE_SIZE // 2, row * SQUARE_SIZE + SQUARE_SIZE // 2), CIRCLE_RADIUS, CIRCLE_WIDTH)

    def draw_cross(self, col, row):
        pygame.draw.line(screen, CROSS_COLOR, (col * SQUARE_SIZE + SPACE, row * SQUARE_SIZE + SQUARE_SIZE - SPACE), 
                         (col * SQUARE_SIZE + SQUARE_SIZE - SPACE, row * SQUARE_SIZE + SPACE), CROSS_WIDTH)
        pygame.draw.line(screen, CROSS_COLOR, (col * SQUARE_SIZE + SPACE, row * SQUARE_SIZE + SPACE), 
                         (col * SQUARE_SIZE + SQUARE_SIZE - SPACE, row * SQUARE_SIZE + SQUARE_SIZE - SPACE), CROSS_WIDTH)

    def reset(self):
        self.board = ['' for _ in range(9)]
        self.draw_lines()

# Game class to manage the game state and logic
class Game:
    def __init__(self):
        self.board = Board()
        self.player = 'X'
        self.ai = 'O'
        self.current_player = self.player
        self.running = True

    def check_winner(self):
        winning_combinations = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],  # Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8],  # Columns
            [0, 4, 8], [2, 4, 6]  # Diagonals
        ]
        for combo in winning_combinations:
            if self.board.board[combo[0]] == self.board.board[combo[1]] == self.board.board[combo[2]] and self.board.board[combo[0]] != '':
                return self.board.board[combo[0]]
        return None

    def is_board_full(self):
        return all([spot != '' for spot in self.board.board])

    def get_available_moves(self):
        return [i for i, spot in enumerate(self.board.board) if spot == '']

    def minimax(self, board, depth, is_maximizing):
        winner = self.check_winner()
        if winner == self.ai:
            return 1
        elif winner == self.player:
            return -1
        elif self.is_board_full():
            return 0

        if is_maximizing:
            best_score = float('-inf')
            for move in self.get_available_moves():
                board[move] = self.ai
                score = self.minimax(board, depth + 1, False)
                board[move] = ''
                best_score = max(score, best_score)
            return best_score
        else:
            best_score = float('inf')
            for move in self.get_available_moves():
                board[move] = self.player
                score = self.minimax(board, depth + 1, True)
                board[move] = ''
                best_score = min(score, best_score)
            return best_score

    def get_best_move(self):
        best_score = float('-inf')
        best_move = None
        for move in self.get_available_moves():
            self.board.board[move] = self.ai
            score = self.minimax(self.board.board, 0, False)
            self.board.board[move] = ''
            if score > best_score:
                best_score = score
                best_move = move
        return best_move

    def make_move(self, index):
        if self.board.board[index] == '' and self.running:
            self.board.board[index] = self.current_player
            self.board.draw_figures()
            pygame.display.update()
            winner = self.check_winner()
            if winner or self.is_board_full():
                self.running = False
                self.display_winner(winner)
            self.current_player = self.ai if self.current_player == self.player else self.player

    def display_winner(self, winner):
        font = pygame.font.Font(None, 54)
        if winner:
            text = font.render(f"{winner} wins!", True, RED)
        else:
            text = font.render("It's a tie!", True, RED)
        screen.blit(text, (WIDTH // 2 - text.get_width() // 2, HEIGHT // 2 - text.get_height() // 2))

    def restart(self):
        self.board.reset()
        self.current_player = self.player
        self.running = True

# Main loop
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Tic Tac Toe with AI')

game = Game()
retry_button = pygame.Rect(WIDTH // 2 - 50, HEIGHT - 50, 100, 40)

def draw_retry_button():
    pygame.draw.rect(screen, BUTTON_COLOR, retry_button)
    font = pygame.font.Font(None, 36)
    text = font.render("Retry", True, BUTTON_TEXT_COLOR)
    screen.blit(text, (retry_button.x + 10, retry_button.y + 5))

game.board.draw_lines()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.MOUSEBUTTONDOWN and game.running:
            mouseX = event.pos[0]
            mouseY = event.pos[1]
            if mouseY < HEIGHT - 100:  # Ensure click is within game board
                clicked_row = mouseY // SQUARE_SIZE
                clicked_col = mouseX // SQUARE_SIZE
                index = clicked_row * BOARD_COLS + clicked_col
                game.make_move(index)
                if game.current_player == game.ai and game.running:
                    ai_move = game.get_best_move()
                    if ai_move is not None:
                        game.make_move(ai_move)
        if event.type == pygame.MOUSEBUTTONDOWN and not game.running:
            if retry_button.collidepoint(event.pos):
                game.restart()

    game.board.draw_figures()
    if not game.running:
        draw_retry_button()
    pygame.display.update()

