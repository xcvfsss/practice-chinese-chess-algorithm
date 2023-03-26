
import time
import random

# 控制台打印的时候用于刷新的函数
def clear_console():
    print("\n"*100)

# 构建棋子类
class Piece:
    # 创建对象的时候初始化棋子
    def __init__(self,name,position,color):
        self.name=name
        self.position=position
        self.color=color
    # 用于print的时候打印对象
    def __str__(self):
        # 如果这个棋子是红色的，返回代表棋子英文名称的大写
        if self.color=='red':
            return self.name.upper()
        else:
            return self.name.lower()

# 构建具体各个棋子的子类

# 兵
class Soldier(Piece):
    def __init__(self,position,color):
        super().__init__('S',position,color)

    def can_move(self,new_position,board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y
        if self.color=='red':
            if dx==-1 and dy==0:
                return True
            if x<5 and abs(dy)==1 and dx==0:
                return True
        else:
            if dx==1 and dy==0:
                return True
            if x>4 and abs(dy)==1 and dx==0:
                return True
        return False
# 炮
class Cannon(Piece):
    def __init__(self,position,color):
        super().__init__('C',position,color)

    def can_move(self, new_position, board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y
        if dx == 0 and dy != 0:
            step = 1 if dy > 0 else -1
            y += step
            cnt = 0
            while y != new_y:
                if board.board[x][y] is not None:
                    cnt += 1
                y += step
            if board.board[new_x][new_y] is None:
                return cnt == 0
            else:
                return cnt == 1
        elif dy == 0 and dx != 0:
            step = 1 if dx > 0 else -1
            x += step
            cnt = 0
            while x != new_x:
                if board.board[x][y] is not None:
                    cnt += 1
                x += step
            if board.board[new_x][new_y] is None:
                return cnt == 0
            else:
                return cnt == 1
# 车
class Rook(Piece):
    def __init__(self, position, color):
        super().__init__('R', position, color)

    def can_move(self, new_position, board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y

        if dx == 0 and dy != 0:
            step = 1 if dy > 0 else -1
            y += step
            while y != new_y:
                if board.board[x][y] is not None:
                    return False
                y += step
            return True
        elif dy == 0 and dx != 0:
            step = 1 if dx > 0 else -1
            x += step
            while x != new_x:
                if board.board[x][y] is not None:
                    return False
                x += step
            return True

        return False
# 马
class Knight(Piece):
    def __init__(self,position,color):
        super().__init__('M',position,color)

    def can_move(self, new_position, board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y
        if abs(dx) == 2 and abs(dy)== 1:
            if board.board[x + (dx // 2)][y] is None:
                return True
        elif abs(dy) == 2 and abs(dx) == 1:
            if board.board[x][y + (dy //2)] is None:
                return True
        return False
# 象
class Bishop(Piece):
    def __init__(self,position,color):
        super().__init__('B',position,color)

    def can_move(self, new_position, board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y
        if abs(dx) == 2 and abs(dy) ==2:
            if self.color == 'red' and new_x < 5:
                return False
            if self.color == 'black' and new_x > 4:
                return False
            if board.board[x + dx // 2][y + dy // 2] is None:
                return True
        return False
# 士
class Guard(Piece):
    def __init__(self,position,color):
        super().__init__('G',position,color)

    def can_move(self, new_position, board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y
        if abs(dx) == 1 and abs(dy) == 1:
            if self.color == 'red' and (new_x < 7 or new_x > 9):
                return False
            if self.color == 'black' and (new_x <0 or new_x > 2):
                return False
            if new_y < 3 or new_y > 5:
                return False
            return True
        return False
# 将帅
class King(Piece):
    def __init__(self,position,color):
        super().__init__('K',position,color)

    def can_move(self, new_position, board):
        x, y = self.position
        new_x, new_y = new_position
        dx, dy = new_x - x, new_y - y
        if self.color == 'red' and (new_x < 7 or new_x > 9):
            return False
        if self.color == 'black' and (new_x < 0 or new_x > 2):
            return False
        if new_y < 3 or new_y > 5:
            return False
        if (abs(dx) == 1 and dy == 0) or (dx == 0 and abs(dy) == 1):
            if self.is_facing_king_without_piece(new_position,board):
                return False
            return True
        return False

    def is_facing_king_without_piece(self, new_position, board):
        x, y = new_position
        dx = -1 if self.color == 'red' else 1
        x += dx
        while 0 <= x < 10:
            piece = board.board[x][y]
            if piece is not None:
                if piece.name == 'K' and piece.color != self.color:
                    return True
                break
            x += dx
        return False


# 构建棋盘类
class Board:
    def __init__(self):
        self.board=self.init_board()

    # 初始化棋盘
    def init_board(self):
        board=[[None for _ in range(9)] for _ in range(10)]

        board[0][0] = Rook((0, 0), 'black')
        board[0][1] = Knight((0, 1), 'black')
        board[0][2] = Bishop((0, 2), 'black')
        board[0][3] = Guard((0, 3), 'black')
        board[0][4] = King((0, 4), 'black')
        board[0][5] = Guard((0, 5), 'black')
        board[0][6] = Bishop((0, 6), 'black')
        board[0][7] = Knight((0, 7), 'black')
        board[0][8] = Rook((0, 8), 'black')
        board[2][1] = Cannon((2, 1), 'black')
        board[2][7] = Cannon((2, 7), 'black')
        for i in range(9):
            if i%2==0:
                board[3][i]=Soldier((3,i),'black')

        board[9][0] = Rook((9, 0), 'red')
        board[9][1] = Knight((9, 1), 'red')
        board[9][2] = Bishop((9, 2), 'red')
        board[9][3] = Guard((9, 3), 'red')
        board[9][4] = King((9, 4), 'red')
        board[9][5] = Guard((9, 5), 'red')
        board[9][6] = Bishop((9, 6), 'red')
        board[9][7] = Knight((9, 7), 'red')
        board[9][8] = Rook((9, 8), 'red')
        board[7][1] = Cannon((7, 1), 'red')
        board[7][7] = Cannon((7, 7), 'red')
        for i in range(9):
            if i%2==0:
                board[6][i]=Soldier((6,i),'red')
        return board

    # 用于棋子的映射
    UNICODE_PIECES = {
        'R': '車 ', 'r': '车1',
        'M': '馬 ', 'm': '马1',
        'B': '象 ', 'b': '相1',
        'G': '仕 ', 'g': '士1',
        'K': '帥 ', 'k': '将1',
        'C': '炮 ', 'c': '砲1',
        'S': '兵 ', 's': '卒1'
    }

    # 用函数打印棋盘
    def display(self):
        for row in self.board:
            row_str=""
            for piece in row:
                if piece is not None:
                    cell_str=Board.UNICODE_PIECES[str(piece)]
                else:
                    cell_str='。'
                row_str+="{:^5}".format(cell_str)
            print(row_str)

    # 用于print的时候打印棋盘对象
    def __str__(self):
        result=[]
        for row in self.board:
            row_str=''
            for piece in row:
                if piece is not None:
                    cell_str=Board.UNICODE_PIECES[str(piece)]
                else :
                    cell_str='。'
                row_str+="{:<5}".format(cell_str)
            result.append(row_str)
        return "\n\n".join(result)

    # 用于棋子的移动
    def move(self,src_position,dst_position):
        x,y=src_position
        new_x,new_y=dst_position
        piece=self.board[x][y]
        if piece is not None and piece.can_move(dst_position,self):
            target=self.board[new_x][new_y]
            if target is None or target.color!=piece.color:
                self.board[new_x][new_y]=piece
                self.board[x][y]=None
                piece.position=dst_position
                return True
        return False

    # 检查棋子是否合理移动
    def is_valid_move(self,src_position,dst_position):
        x,y=src_position
        new_x,new_y=dst_position
        piece=self.board[x][y]
        if piece is not None and piece.can_move(dst_position,self):
            target=self.board[new_x][new_y]
            if target is None or target.color!=piece.color:
                return True
        return False


# 构建随机落子的程序
class RandomAI:
    def __init__(self,game):
        self.game=game
    def random_black_move(self):
        # 将所有的黑色棋子的当前位置加入pieces列表中
        pieces=[]
        for i in range(10):
            for j in range(9):
                piece=self.game.board.board[i][j]
                if piece is not None and piece.color=='black':
                    pieces.append((i,j))
        # 如果pieces列表为空的，则表明棋盘上没有黑色棋子了，于是直接返回
        if len(pieces)==0:
            return
        # 从列表中随机取出一个棋子
        piece=random.choice(pieces)

        # 将随机取出的棋子的所有可移动到的位置添加到moves列表中
        # 如果这个棋子没有可以移动的位置，黑方就不动了。那么就要重新挑选棋子

        while len(pieces) != 0:
            moves = []
            for i in range(10):
                for j in range(9):
                    if self.game.board.is_valid_move(piece, (i, j)):
                        moves.append((piece,(i,j)))
                        self.game.board.move((i,j),piece)
            if len(moves)==0:
                pieces.remove(piece)
                piece=random.choice(pieces)
                continue
                # 重新挑选棋子，并将前面一个棋子从pieces中删除，然后重新寻找这个棋子的所有可移动的位置
            else:
                break

        second_elements = [tup[1] for tup in moves]
        print(second_elements)
        print("输出pieces列表：pieces:", pieces)  # 添加调试语句，输出pieces列表
        print("输出选中的棋子：selected piece:", piece)  # 添加调试语句，输出选中的棋子
        print("输出可移动位置列表：valid moves:", second_elements)  # 添加调试语句，输出可移动位置列表

        move = random.choice(moves)
        print("输出选中的落子位置：selected move:", move)  # 添加调试语句，输出选中的落子位置

        self.game.board.move(move[0],move[1])

    def random_red_move(self):
        # 将所有的黑色棋子的当前位置加入pieces列表中
        pieces=[]
        for i in range(10):
            for j in range(9):
                piece=self.game.board.board[i][j]
                if piece is not None and piece.color=='red':
                    pieces.append((i,j))
        # 如果pieces列表为空的，则表明棋盘上没有黑色棋子了，于是直接返回
        if len(pieces) == 0:
            return
        # 从列表中随机取出一个棋子
        piece = random.choice(pieces)

        # 将随机取出的棋子的所有可移动到的位置添加到moves列表中
        # 如果这个棋子没有可以移动的位置，黑方就不动了。那么就要重新挑选棋子

        while len(pieces) != 0:
            moves = []
            for i in range(10):
                for j in range(9):
                    if self.game.board.is_valid_move(piece, (i, j)):
                        moves.append((piece, (i, j)))
                        self.game.board.move((i, j), piece)
            if len(moves) == 0:
                pieces.remove(piece)
                piece = random.choice(pieces)
                continue
                # 重新挑选棋子，并将前面一个棋子从pieces中删除，然后重新寻找这个棋子的所有可移动的位置
            else:
                break
        second_elements = [tup[1] for tup in moves]
        print(second_elements)
        print("输出pieces列表：pieces:", pieces)  # 添加调试语句，输出pieces列表
        print("输出选中的棋子：selected piece:", piece)  # 添加调试语句，输出选中的棋子
        print("输出可移动位置列表：valid moves:", second_elements)  # 添加调试语句，输出可移动位置列表

        move = random.choice(moves)
        print("输出选中的落子位置：selected move:", move)  # 添加调试语句，输出选中的落子位置
        self.game.board.move(move[0],move[1])

    def play(self):
        while True:
            # clear_console()
            print(self.game.board)
            if self.game.turn=='red':
                input('下面进入红方回合，按1确认:')
                # time.sleep(1)
                self.random_red_move()
                self.game.turn='black'
            else :
                input('下面进入黑方回合，按1确认:')
                # time.sleep(1)
                self.random_black_move()
                self.game.turn='red'

# 构建游戏类，首先要有一个游戏，游戏里有棋盘，棋盘上有棋子，棋子有很多种，每种棋子有不同的走法
class Game:
    def __init__(self):
        self.board=Board()
        self.turn='red'

    def play(self):
        while True:
            pass

if __name__ == '__main__':
    game=Game()
    RanDom_MY_AI=RandomAI(game)
    RanDom_MY_AI.play()
    # board=Board()
    # print(board)