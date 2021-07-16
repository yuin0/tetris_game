#!/usr/bin/python3
# -*- coding: utf-8 -*-

from datetime import datetime  # To pick up date
from operator import mul  # To multiple lists
import pprint  # To customize output
import copy  # To copy objects
# "copy" does not copy object's contents:
# the copy has contents referencing to the same of copied one.
# "deepCopy" copies object's contents too:
# the copy has contents that does not reference to the copied one.


class Block_Controller(object):  # object is not necessary (to use python2)

    # init parameter
    board_backboard = 0
    board_data_width = 0
    board_data_height = 0
    ShapeNone_index = 0
    CurrentShape_class = 0
    NextShape_class = 0

    # GetNextMove is main function.
    # input
    #    nextMove : nextMove structure which is empty.
    #    GameStatus : block/field/judge/debug information.
    #                 in detail see the internal GameStatus data. @game_manager
    # output
    #    nextMove : nextMove structure which includes next shape position.
    def GetNextMove(self, nextMove, GameStatus):

        t1 = datetime.now()

        # print GameStatus
        print("=================================================>")
        pprint.pprint(GameStatus, width=61, compact=True)

        # get data from GameStatus
        # current shape info
        CurrentShapeDirectionRange\
            = GameStatus["block_info"]["currentShape"]["direction_range"]
        # reference dictionary type list
        self.CurrentShape_class\
            = GameStatus["block_info"]["currentShape"]["class"]
        # "self.~" means this class elements
        # next shape info
        NextShapeDirectionRange\
            = GameStatus["block_info"]["nextShape"]["direction_range"]
        self.NextShape_class\
            = GameStatus["block_info"]["nextShape"]["class"]
        # current board info
        self.board_backboard = GameStatus["field_info"]["backboard"]
        # default board definition
        self.board_data_width = GameStatus["field_info"]["width"]
        self.board_data_height = GameStatus["field_info"]["height"]
        self.ShapeNone_index\
            = GameStatus["debug_info"]["shape_info"]["shapeNone"]["index"]

        # search best nextMove -->
        strategy = None  # Initialize
        LatestEvalValue = -100000
        nowFL = 0
        nextFL = 0
        # search with current block Shape
        for direction0 in CurrentShapeDirectionRange:
            # search with x range
            x0Min, x0Max\
                = self.getSearchXRange(self.CurrentShape_class, direction0)
            for x0 in range(x0Min, x0Max):
                # get board data, as if dropdown block
                board = self.getBoard(self.board_backboard,
                                      self.CurrentShape_class, direction0, x0)
                board, offsetFL = self.getBoardWithoutFL(board)

                for direction1 in NextShapeDirectionRange:
                    x1Min, x1Max = self.getSearchXRange(self.NextShape_class,
                                                        direction1)
                    for x1 in range(x1Min, x1Max):
                        # get next board Data
                        board1 = self.getBoard(board, self.NextShape_class,
                                               direction1, x1)
                        board1, fullLines = self.getBoardWithoutFL(board1)
                        # evaluate board
                        EvalValue = self.calcEvaluationValue(board1, offsetFL,
                                                             fullLines)
                        # update best move
                        if EvalValue > LatestEvalValue:
                            strategy = (direction0, x0, 1, 1)
                            LatestEvalValue = EvalValue
                            nowFL = offsetFL
                            nextFL = fullLines

        print("===", datetime.now() - t1)
        nextMove["strategy"]["direction"] = strategy[0]
        nextMove["strategy"]["x"] = strategy[1]
        nextMove["strategy"]["y_operation"] = strategy[2]
        nextMove["strategy"]["y_moveblocknum"] = strategy[3]
        print(nextMove)
        print("Evaluation: ", LatestEvalValue)
        print("nowFL: ", nowFL, "nextFL: ", nextFL)
        print("###### SAMPLE CODE ######")
        return nextMove

    def getSearchXRange(self, Shape_class, direction):
        #
        # get x range from shape direction.
        # get shape x offsets[minX,maxX] as relative value.
        minX, maxX, _, _ = Shape_class.getBoundingOffsets(direction)
        xMin = -1 * minX
        xMax = self.board_data_width - maxX
        return xMin, xMax

    def getShapeCoordArray(self, Shape_class, direction, x, y):
        #
        # get coordinate array by given shape.
        #
        # get array from shape direction, x, y.
        coordArray = Shape_class.getCoords(direction, x, y)
        return coordArray

    def getBoard(self, board_backboard, Shape_class, direction, x):
        #
        # get new board.
        #
        # copy backboard data to make new board.
        # if not, original backboard data will be updated later.
        board = copy.deepcopy(board_backboard)
        _board = self.dropDown(board, Shape_class, direction, x)
        return _board

    def dropDown(self, board, Shape_class, direction, x):
        #
        # internal function of getBoard.
        # -- drop down the shape on the board.
        #
        dy = self.board_data_height - 1
        coordArray = self.getShapeCoordArray(Shape_class, direction, x, 0)
        # update dy
        for _x, _y in coordArray:
            _yy = 0
            while _yy + _y < self.board_data_height \
                    and (_yy + _y < 0
                         or board[(_y + _yy) * self.board_data_width + _x]
                         == self.ShapeNone_index):
                _yy += 1
            _yy -= 1
            if _yy < dy:
                dy = _yy
        # get new board
        _board = self.dropDownWithDy(board, Shape_class, direction, x, dy)
        return _board

    def dropDownWithDy(self, board, Shape_class, direction, x, dy):
        #
        # internal function of dropDown.
        #
        _board = board
        coordArray = self.getShapeCoordArray(Shape_class, direction, x, 0)
        for _x, _y in coordArray:
            _board[(_y + dy) * self.board_data_width + _x] = Shape_class.shape
        return _board

    def getBoardWithoutFL(self, board):
        width = self.board_data_width
        height = self.board_data_height
        nextBoard = [0] * width * height
        newY = height - 1
        fullLines = 0
        for y in range(height - 1, -1, -1):
            blockCount = sum([1 if board[x + y * width] > 0 else 0
                             for x in range(width)])
            if not blockCount < width:
                fullLines += 1
                continue
            for x in range(width):
                nextBoard[x + newY * width] = board[x + y * width]
            newY -= 1
        return nextBoard, fullLines

    def calcEvaluationValue(self, board, offsetFL, fullLines):
        #
        # sample function of evaluate board.
        #
        width = self.board_data_width
        height = self.board_data_height

        # evaluation paramters
        # number of holes or blocks in the line.
        nHoles = 0
        # nIsolatedBlocks = 0
        # absolute differencial value of MaxY
        absDy = 0
        # how blocks are accumlated
        BlockMaxY = [0] * width
        holeCandidates = [0] * width
        holeConfirm = [0] * width
        # isolated blocks of x
        isolatedBlocks = [0] * width
        # the highest hole of x
        holeMaxY = [0] * width
        # number of horizontal changes

        # check board
        # each y line
        # range(start, stop, step) from top line to bottom line.
        for y in range(height - 1, 0, -1):
            hasHole = False
            hasBlock = False
            # each x line
            for x in range(width):
                # check if hole or block.
                if board[y * width + x] == self.ShapeNone_index:
                    # ShapeNone=0, so serach points printing "0".
                    # hole
                    hasHole = True
                    holeCandidates[x] += 1  # just candidates in each column..
                else:
                    # block
                    hasBlock = True
                    BlockMaxY[x] = height - y  # update blockMaxY
                    if holeCandidates[x] > 0:
                        # update number of holes in target column
                        holeConfirm[x] += holeCandidates[x]
                        holeCandidates[x] = 0  # reset.
                        holeMaxY[x] = height - y - 1  # update the highest hole
                    if holeConfirm[x] > 0:
                        # update number of isolated blocks.
                        # if hole exits,isolatedBlock also exists.
                        isolatedBlocks[x] += 1
            # at least one hole exists, hasHole will be true.
            if hasBlock and not hasHole:
                # filled with block
                fullLines += 1
                # the line to be checked, for there are both blocks and holes.
            elif hasBlock and hasHole:
                pass
            elif not hasBlock:
                # no block line (and ofcourse no hole)
                pass

        # nHoles
        # holeConfirm is an Array whose item is holes of each x.
        for x in holeConfirm:
            nHoles += abs(x)

        # maxHeight
        maxHeight = max(BlockMaxY) - fullLines

        # absolute differencial value of MaxY
        BlockMaxDy = []
        for i in range(len(BlockMaxY) - 1):
            val = BlockMaxY[i] - BlockMaxY[i+1]
            BlockMaxDy += [val]
        for x in BlockMaxDy:
            absDy += abs(x)
        if BlockMaxDy[0] < -3 and BlockMaxDy[-1] > 3:
            absDy += 3 * (abs(BlockMaxDy[0]) + abs(BlockMaxDy[-1]))

        # isolatedBlocksPenalty
        onHolePenalty = sum(map(mul, isolatedBlocks, holeMaxY))

        maxDy = max(BlockMaxY) - sorted(BlockMaxY)[2]

        # statistical data
        # stdY
        # if len(BlockMaxY) <= 0:
        #    stdY = 0
        # else:
        #    stdY = math.sqrt(sum([y ** 2 for y in BlockMaxY]) / len(BlockMaxY)
        #                       - (sum(BlockMaxY) / len(BlockMaxY)) ** 2)
        # stdDY
        # if len(BlockMaxDy) <= 0:
        #    stdDY = 0
        # else:
        #    stdDY = math.sqrt(sum([y ** 2 for y in BlockMaxDy])
        #                       /len(BlockMaxDy)
        #                      - (sum(BlockMaxDy) / len(BlockMaxDy)) ** 2)

        # calc Evaluation Value
        score = 0
        if fullLines == 4:
            score = score + fullLines * 100
        elif fullLines > 0:
            if maxHeight < 9:
                score = score - 9
            else:
                score = score + fullLines
        if offsetFL == 4:
            score = score + offsetFL * 100
        elif fullLines < 4 and offsetFL > 0:
            if maxHeight < 9:
                score = score - 9
            else:
                score = score + offsetFL
        score = score - nHoles * 10.0  # try not to make hole
        score = score - onHolePenalty * 0.3
        score = score - absDy * 1.0                 # try to put block smoothly
        if maxHeight > 12:
            score = score - maxHeight * 5              # maxHeight
        score = score - maxDy * 1.0
        # score = score - stdY * 1.0                 # statistical data
        # score = score - stdDY * 0.01               # statistical data

        # print(score, fullLines, nHoles, nIsolatedBlocks,
        #       maxHeight, stdY, stdDY, absDy, BlockMaxY)
        return score


BLOCK_CONTROLLER = Block_Controller()
