import math

# ---------- CONFIGURATION ----------
HUMAN = "X"          # Mark used by the human
AI     = "O"         # Mark used by the computer
EMPTY  = " "         # Blank cell

# ---------- GAME LOGIC -------------
WIN_PATTERNS = [
    (0, 1, 2), (3, 4, 5), (6, 7, 8),      # Rows
    (0, 3, 6), (1, 4, 7), (2, 5, 8),      # Columns
    (0, 4, 8), (2, 4, 6)                  # Diagonals
]

def print_board(board):
    """Pretty-prints the current board."""
    def _row(start):
        return f" {board[start]} | {board[start+1]} | {board[start+2]} "
    divider = "\n---+---+---\n"
    print(_row(0) + divider + _row(3) + divider + _row(6))

def available_moves(board):
    """Returns a list of indices for empty squares."""
    return [i for i, v in enumerate(board) if v == EMPTY]

def is_winner(board, player):
    """Checks whether 'player' has a winning pattern."""
    return any(all(board[i] == player for i in pattern) for pattern in WIN_PATTERNS)

def is_terminal(board):
    """Returns (True, score) if the board is a terminal state."""
    if is_winner(board, AI):
        return True,  1      # AI won
    if is_winner(board, HUMAN):
        return True, -1      # Human won
    if EMPTY not in board:
        return True,  0      # Draw
    return False, None

# ---------- MINIMAX WITH ALPHA-BETA ----------
def minimax(board, depth, is_maximizing, alpha=-math.inf, beta=math.inf):
    terminal, score = is_terminal(board)
    if terminal:
        return score

    if is_maximizing:                       # AI's turn
        max_eval = -math.inf
        for move in available_moves(board):
            board[move] = AI
            eval = minimax(board, depth + 1, False, alpha, beta)
            board[move] = EMPTY
            max_eval = max(max_eval, eval)
            alpha = max(alpha, eval)
            if beta <= alpha:
                break                       # Beta cut-off
        return max_eval
    else:                                   # Human's turn
        min_eval = math.inf
        for move in available_moves(board):
            board[move] = HUMAN
            eval = minimax(board, depth + 1, True, alpha, beta)
            board[move] = EMPTY
            min_eval = min(min_eval, eval)
            beta = min(beta, eval)
            if beta <= alpha:
                break                       # Alpha cut-off
        return min_eval

def best_move(board):
    """Finds the move with the highest Minimax score for the AI."""
    best_score = -math.inf
    move_choice = None
    for move in available_moves(board):
        board[move] = AI
        score = minimax(board, 0, False, -math.inf, math.inf)
        board[move] = EMPTY
        if score > best_score:
            best_score = score
            move_choice = move
    return move_choice

# ---------- GAME LOOP ---------------
def human_turn(board):
    while True:
        try:
            user_input = input("Choose your move (1-9): ")
            idx = int(user_input) - 1
            if idx in available_moves(board):
                board[idx] = HUMAN
                break
            print("Invalid move, try again.")
        except (ValueError, IndexError):
            print("Please enter a number from 1 to 9 corresponding to an empty cell.")

def play():
    board = [EMPTY] * 9
    print("You are X, AI is O. Cells are numbered as follows:\n")
    print(" 1 | 2 | 3 \n---+---+---\n 4 | 5 | 6 \n---+---+---\n 7 | 8 | 9 \n")
    current_player = HUMAN  # Human goes first

    while True:
        if current_player == HUMAN:
            human_turn(board)
        else:
            move = best_move(board)
            board[move] = AI
            print(f"\nAI chooses cell {move + 1}:\n")

        print_board(board)
        terminal, score = is_terminal(board)
        if terminal:
            if score ==  1:
                print("\nAI wins! Better luck next time.")
            elif score == -1:
                print("\nCongratulations, you win!")
            else:
                print("\nIt's a draw.")
            break

        current_player = AI if current_player == HUMAN else HUMAN

if __name__ == "__main__":
    play()
