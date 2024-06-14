# -chess
#practical chess game
import tkinter as tk

class ChessGame:
    def __init__(self):
        self.board = self.create_board()
        self.current_player = 'White'
        self.move_stack = []
        self.castling_rights = {
            'White': {'king_side': True, 'queen_side': True},
            'Black': {'king_side': True, 'queen_side': True}
        }
        self.en_passant_target = None

    def create_board(self):
        board = [[' ' for _ in range(8)] for _ in range(8)]
        for i in range(8):
            board[1][i] = '♙'  # White pawns
            board[6][i] = '♟'  # Black pawns
        board[0] = ['♖', '♘', '♗', '♕', '♔', '♗', '♘', '♖']  # White pieces
        board[7] = ['♜', '♞', '♝', '♛', '♚', '♝', '♞', '♜']  # Black pieces
        return board

    def make_move(self, start, end):
        piece = self.board[start[0]][start[1]]
        captured = self.board[end[0]][end[1]]
        self.move_stack.append((piece, start, end, captured))
        self.board[start[0]][start[1]] = ' '
        self.board[end[0]][end[1]] = piece

        # Handle castling
        if piece == '♔' and abs(start[1] - end[1]) == 2:
            if end[1] == 2:  # Queen-side castling
                self.board[start[0]][0] = ' '  # Clear rook
                self.board[start[0]][3] = '♖'  # Place rook
            elif end[1] == 6:  # King-side castling
                self.board[start[0]][7] = ' '  # Clear rook
                self.board[start[0]][5] = '♖'  # Place rook

        if piece == '♚' and abs(start[1] - end[1]) == 2:
            if end[1] == 2:  # Queen-side castling
                self.board[start[0]][0] = ' '  # Clear rook
                self.board[start[0]][3] = '♜'  # Place rook
            elif end[1] == 6:  # King-side castling
                self.board[start[0]][7] = ' '  # Clear rook
                self.board[start[0]][5] = '♜'  # Place rook

        # Handle en passant capture
        if piece in '♙♟' and end == self.en_passant_target:
            self.board[start[0]][end[1]] = ' '

        # Handle pawn promotion
        if piece == '♙' and end[0] == 7:
            self.board[end[0]][end[1]] = '♕'
        elif piece == '♟' and end[0] == 0:
            self.board[end[0]][end[1]] = '♛'

        # Update en passant target
        self.en_passant_target = (end[0] + (1 if self.current_player == 'White' else -1), end[1]) if piece in '♙♟' and abs(start[0] - end[0]) == 2 else None

        # Update castling rights
        if piece == '♔':
            self.castling_rights['White']['king_side'] = False
            self.castling_rights['White']['queen_side'] = False
        elif piece == '♚':
            self.castling_rights['Black']['king_side'] = False
            self.castling_rights['Black']['queen_side'] = False
        elif piece == '♖' and start == (0, 0):
            self.castling_rights['White']['queen_side'] = False
        elif piece == '♖' and start == (0, 7):
            self.castling_rights['White']['king_side'] = False
        elif piece == '♜' and start == (7, 0):
            self.castling_rights['Black']['queen_side'] = False
        elif piece == '♜' and start == (7, 7):
            self.castling_rights['Black']['king_side'] = False

        self.switch_player()
        print(f"Move made: {piece} from {start} to {end}")

    def undo_move(self):
        if self.move_stack:
            piece, start, end, captured = self.move_stack.pop()
            self.board[start[0]][start[1]] = piece
            self.board[end[0]][end[1]] = captured
            self.switch_player()
            print(f"Move undone: {piece} from {end} to {start}")

    def switch_player(self):
        self.current_player = 'Black' if self.current_player == 'White' else 'White'
        print(f"Current player: {self.current_player}")

    def is_valid_move(self, start, end):
        piece = self.board[start[0]][start[1]]
        target = self.board[end[0]][end[1]]
        if piece == ' ':
            return False
        if self.is_white_piece(piece) and self.current_player != 'White':
            return False
        if self.is_black_piece(piece) and self.current_player != 'Black':
            return False
        if target != ' ' and ((self.is_white_piece(target) and self.current_player == 'White') or (self.is_black_piece(target) and self.current_player == 'Black')):
            return False
        # Movement rules for each piece
        if piece in '♙♟':  # Pawns
            return self.is_valid_pawn_move(start, end)
        elif piece in '♖♜':  # Rooks
            return self.is_valid_rook_move(start, end)
        elif piece in '♘♞':  # Knights
            return self.is_valid_knight_move(start, end)
        elif piece in '♗♝':  # Bishops
            return self.is_valid_bishop_move(start, end)
        elif piece in '♕♛':  # Queens
            return self.is_valid_queen_move(start, end)
        elif piece in '♔♚':  # Kings
            return self.is_valid_king_move(start, end)
        return False

    def is_valid_pawn_move(self, start, end):
        direction = 1 if self.current_player == 'White' else -1
        start_row, start_col = start
        end_row, end_col = end
        if start_col == end_col:
            if self.board[end_row][end_col] == ' ':
                if end_row - start_row == direction:
                    return True
                if (start_row == 1 and self.current_player == 'White' or start_row == 6 and self.current_player == 'Black') and end_row - start_row == 2 * direction:
                    return self.board[start_row + direction][start_col] == ' '
        elif abs(start_col - end_col) == 1 and end_row - start_row == direction:
            return self.board[end_row][end_col] != ' ' or end == self.en_passant_target
        return False

    def is_valid_rook_move(self, start, end):
        start_row, start_col = start
        end_row, end_col = end
        if start_row != end_row and start_col != end_col:
            return False
        step = 1 if end_row > start_row or end_col > start_col else -1
        if start_row == end_row:
            for col in range(start_col + step, end_col, step):
                if self.board[start_row][col] != ' ':
                    return False
        else:
            for row in range(start_row + step, end_row, step):
                if self.board[row][start_col] != ' ':
                    return False
        return True

    def is_valid_knight_move(self, start, end):
        start_row, start_col = start
        end_row, end_col = end
        return (abs(start_row - end_row), abs(start_col - end_col)) in [(2, 1), (1, 2)]

    def is_valid_bishop_move(self, start, end):
        start_row, start_col = start
        end_row, end_col = end
        if abs(start_row - end_row) != abs(start_col - end_col):
            return False
        row_step = 1 if end_row > start_row else -1
        col_step = 1 if end_col > start_col else -1
        for i, j in zip(range(start_row + row_step, end_row, row_step), range(start_col + col_step, end_col, col_step)):
            if self.board[i][j] != ' ':
                return False
        return True

    def is_valid_queen_move(self, start, end):
        return self.is_valid_rook_move(start, end) or self.is_valid_bishop_move(start, end)

    def is_valid_king_move(self, start, end):
        if max(abs(start[0] - end[0]), abs(start[1] - end[1])) == 1:
            return True
        # Castling
        if start[0] == end[0] and abs(start[1] - end[1]) == 2:
            if self.current_player == 'White' and self.castling_rights['White']['king_side'] and end == (0, 6) and all(self.board[0][i] == ' ' for i in range(5, 7)) and not self.is_check():
                return True
            if self.current_player == 'White' and self.castling_rights['White']['queen_side'] and end == (0, 2) and all(self.board[0][i] == ' ' for i in range(1, 4)) and not self.is_check():
                return True
            if self.current_player == 'Black' and self.castling_rights['Black']['king_side'] and end == (7, 6) and all(self.board[7][i] == ' ' for i in range(5, 7)) and not self.is_check():
                return True
            if self.current_player == 'Black' and self.castling_rights['Black']['queen_side'] and end == (7, 2) and all(self.board[7][i] == ' ' for i in range(1, 4)) and not self.is_check():
                return True
        return False

    def is_check(self):
        king_position = self.find_king_position()
        if king_position is None:
            return False
        for i in range(8):
            for j in range(8):
                if self.board[i][j] != ' ' and ((self.is_black_piece(self.board[i][j]) and self.current_player == 'White') or (self.is_white_piece(self.board[i][j]) and self.current_player == 'Black')):
                    if self.is_valid_move((i, j), king_position):
                        return True
        return False

    def is_checkmate(self):
        if not self.is_check():
            return False
        for i in range(8):
            for j in range(8):
                piece = self.board[i][j]
                if piece != ' ' and ((self.is_white_piece(piece) and self.current_player == 'White') or (self.is_black_piece(piece) and self.current_player == 'Black')):
                    for x in range(8):
                        for y in range(8):
                            if self.is_valid_move((i, j), (x, y)):
                                self.make_move((i, j), (x, y))
                                if not self.is_check():
                                    self.undo_move()
                                    return False
                                self.undo_move()
        return True

    def is_white_piece(self, piece):
        return piece in '♙♖♘♗♕♔'

    def is_black_piece(self, piece):
        return piece in '♟♜♞♝♛♚'

    def find_king_position(self):
        king_symbol = '♔' if self.current_player == 'White' else '♚'
        for i in range(8):
            for j in range(8):
                if self.board[i][j] == king_symbol:
                    return (i, j)
        return None

class ChessGameUI(tk.Tk):
    def __init__(self, game):
        super().__init__()
        self.game = game
        self.title("Chess Game")
        self.geometry("400x440")

        self.canvas = tk.Canvas(self, width=400, height=400, bg="white")
        self.canvas.pack()

        self.draw_board()
        self.selected_piece = None
        self.bind("<Button-1>", self.handle_click)

        self.back_button = tk.Button(self, text="Back", command=self.undo_move)
        self.back_button.pack()

    def draw_board(self):
        self.canvas.delete("all")
        for i in range(8):
            for j in range(8):
                color = "white" if (i + j) % 2 == 0 else "gray"
                self.canvas.create_rectangle(j * 50, i * 50, (j + 1) * 50, (i + 1) * 50, fill=color)

        is_check = self.game.is_check()
        for i in range(8):
            for j in range(8):
                piece = self.game.board[i][j]
                if piece != ' ':
                    if piece == '♚' or piece == '♔':
                        color = "red" if is_check and self.is_king_in_check((i, j)) else "black"
                        self.canvas.create_text((j + 0.5) * 50, (i + 0.5) * 50, text=piece, font=("Arial", 24), fill=color)
                    else:
                        self.canvas.create_text((j + 0.5) * 50, (i + 0.5) * 50, text=piece, font=("Arial", 24))

        if self.game.is_checkmate():
            self.canvas.create_text(200, 200, text="Checkmate!", font=("Arial", 24), fill="red")
            self.after(3000, self.quit)  # Close the game after 3 seconds

    def is_king_in_check(self, position):
        king_position = self.game.find_king_position()
        return position == king_position

    def handle_click(self, event):
        col = event.x // 50
        row = event.y // 50
        if self.selected_piece is None:
            piece = self.game.board[row][col]
            if piece != ' ' and ((self.is_white_piece(piece) and self.game.current_player == 'White') or (self.is_black_piece(piece) and self.game.current_player == 'Black')):
                self.selected_piece = (row, col)
                print(f"Selected piece: {piece} at {self.selected_piece}")
        else:
            start_row, start_col = self.selected_piece
            if (row, col) != (start_row, start_col) and self.game.is_valid_move((start_row, start_col), (row, col)):
                self.game.make_move((start_row, start_col), (row, col))
                self.selected_piece = None
            else:
                self.selected_piece = None
            print(f"Move attempted from {start_row, start_col} to {row, col}")
        self.draw_board()
        if self.game.is_checkmate():
            self.after(3000, self.quit)  # Close the game after 3 seconds if checkmate

    def is_white_piece(self, piece):
        return piece in '♙♖♘♗♕♔'

    def is_black_piece(self, piece):
        return piece in '♟♜♞♝♛♚'

    def undo_move(self):
        self.game.undo_move()
        self.draw_board()

    def play(self):
        self.mainloop()

if __name__ == "__main__":
    game = ChessGame()
    ui = ChessGameUI(game)
    ui.play()

