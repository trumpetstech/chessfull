<template>
     <div>
        <div id="chessboard" style="width: 70vh;"></div>
    </div>
</template>


<script>

    var Chess = require('chess.js');
    import ChessBoard from 'chessboardjs';

    export default {
        props: [],

        data() {
            return {
                board: {},
                game: {},
                cfg: {
                    draggable: false,
                    position: 'start',
                    pieceTheme: 'https://chessvicky.com/admin/img/chesspieces/wikipedia/{piece}.png'
                },
                timeOver: false,
                playerColor: 'w',
                source: 0,
                destination: 0,
                positionCount: 0,


            }
        },



        mounted() {

             this.board = new ChessBoard('chessboard', this.cfg);
             this.game = new Chess();
            $("#chessboard").on("click", ".square-55d63", this.clickOnSquare);

        },

        methods: {
            clickOnSquare(evt) {
                var target = evt.currentTarget;

                if(!this.isGameOver())
                {
                    if(this.isPlayerTurn())
                    {
                        var square = $(target).data("square"); // selected square
                        var squareEl = $('#chessboard .square-' + square); // selected square jquery elem
                        $('#chessboard .square-55d63').css('background', ''); // reset all background

                        this.highlightSquare(squareEl);

                        if(!this.inMove())
                        {
                            this.source = square;
                        } else {
                            this.destination = square;

                            this.makeMove();
                        }
                    }
                }


            },

            isGameOver() {
                if (this.game.game_over() || this.timeOver) {
                    return true;
                }
                return false;
            },

            isPlayerTurn() {
                 return this.game.turn() == this.playerColor;
            },

            inMove() {
                return this.source != 0;
            },

            makeMove() {
                var move = this.game.move({
                      from: this.source.toString(),
                      to: this.destination.toString(),
                      promotion: 'q' // NOTE: always promote to a queen for example simplicity
                    });
                 if (move != null) {
                   this.board.position(this.game.fen());
                    //playAudio();
                    this.highlightSquare($('#chessboard .square-' + this.destination));
                    var background = '#a9a9a9';
                    if ($('#chessboard .square-' + this.source).hasClass('black-3c85d') === true) {
                      background = '#696969';
                    }
                    $('#chessboard .square-' + this.source).css('background', background);

                    this.computerMove();
                  //  prepareMove();

                 } else {
                      $('#board .square-55d63').css('background', '');
                 }
                     this.source = 0;

            },

            highlightSquare(square) {
                 var background = '#a9a9a9';
                if (square.hasClass('black-3c85d') === true) {
                  background = '#696969';
                }
                square.css('background', background);
            },

            computerMove() {
                var bestMove = this.getBestMove(this.game);
                this.game.ugly_move(bestMove);
                this.board.position(game.fen());

                if (this.game.game_over()) {
                    alert('Game over');
                }
            },

            /*The "AI" part starts here */

             getBestMove(game) {
                    if (game.game_over()) {
                        alert('Game over');
                    }

                    this.positionCount = 0;
                    var depth = 3;

                    var d = new Date().getTime();
                    var bestMove = this.minimaxRoot(depth, game, true);
                    var d2 = new Date().getTime();
                    var moveTime = (d2 - d);
                    var positionsPerS = ( this.positionCount * 1000 / moveTime);

                    return bestMove;
            },

            minimaxRoot (depth, game, isMaximisingPlayer) {

                var newGameMoves = game.ugly_moves();
                var bestMove = -9999;
                var bestMoveFound;

                for(var i = 0; i < newGameMoves.length; i++) {
                    var newGameMove = newGameMoves[i]
                    game.ugly_move(newGameMove);
                    var value = minimax(depth - 1, game, -10000, 10000, !isMaximisingPlayer);
                    game.undo();
                    if(value >= bestMove) {
                        bestMove = value;
                        bestMoveFound = newGameMove;
                    }
                }
                return bestMoveFound;
            },

            minimax(depth, game, alpha, beta, isMaximisingPlayer) {
                this.positionCount++;
                if (depth === 0) {
                    return -this.evaluateBoard(game.board());
                }

                var newGameMoves = game.ugly_moves();

                if (isMaximisingPlayer) {
                    var bestMove = -9999;
                    for (var i = 0; i < newGameMoves.length; i++) {
                        game.ugly_move(newGameMoves[i]);
                        bestMove = Math.max(bestMove, this.minimax(depth - 1, game, alpha, beta, !isMaximisingPlayer));
                        game.undo();
                        alpha = Math.max(alpha, bestMove);
                        if (beta <= alpha) {
                            return bestMove;
                        }
                    }
                    return bestMove;
                } else {
                    var bestMove = 9999;
                    for (var i = 0; i < newGameMoves.length; i++) {
                        game.ugly_move(newGameMoves[i]);
                        bestMove = Math.min(bestMove, this.cminimax(depth - 1, game, alpha, beta, !isMaximisingPlayer));
                        game.undo();
                        beta = Math.min(beta, bestMove);
                        if (beta <= alpha) {
                            return bestMove;
                        }
                    }
                    return bestMove;
                }
            },

            evaluateBoard(board) {
                var totalEvaluation = 0;
                for (var i = 0; i < 8; i++) {
                    for (var j = 0; j < 8; j++) {
                        totalEvaluation = totalEvaluation + getPieceValue(board[i][j], i ,j);
                    }
                }
                return totalEvaluation;
            },

            getPieceValue(piece, x, y) {
                if (piece === null) {
                    return 0;
                }
                var getAbsoluteValue = function (piece) {
                    if (piece.type === 'p') {
                        return 10;
                    } else if (piece.type === 'r') {
                        return 50;
                    } else if (piece.type === 'n') {
                        return 30;
                    } else if (piece.type === 'b') {
                        return 30;
                    } else if (piece.type === 'q') {
                        return 90;
                    } else if (piece.type === 'k') {
                        return 900;
                    }
                    throw "Unknown piece type: " + piece.type;
                };

                var absoluteValue = getAbsoluteValue(piece);
                return piece.color === 'w' ? absoluteValue : -absoluteValue;
            },








        }
    }
</script>

