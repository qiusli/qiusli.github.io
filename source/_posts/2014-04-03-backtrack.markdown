---
layout: post
title: "使用递归计算的回溯法"
date: 2014-04-03 21:59
comments: true
categories: 
---
谈到这个方法的初衷本来是想在iOS项目中使用,因为当时叫设计一种方法来摆放battleship,如果在当前位置不能摆放可以回溯到上一个地点再试,最后还是放弃了,因为用不了这么复杂的方法. 不过既然学习了,并且花了这么多时间来编码,就应该记录下来,以后可能还会用上,就不必又从开头来学了.

###什么是回溯法###
回溯法就是先在一条路上走到黑,发现还没达到目的,然后就退回到上一步,然后把另一个发展方向试一遍,直到找到一条通路为止.

<!--more-->

###回溯法的应用###
回溯法最经典的应用就是黑八皇后问题,他要求八颗棋子分别代表八个皇后,然后要把这八个皇后分别放在棋盘上,并且满足如下条件: 

* 任意两个皇后不能在同一行
* 任意两个皇后不能在同一列
* 任意两个皇后不能在同一条斜线上

###代码###
####避免冲突####
下面的代码是用来避免冲突的,即满足上面列出的三个条件.

```java
private boolean threaten(int row, int column) {
        int rowT = row;
        int columnT = column;
        // same row check
        for (int i = 0; i < boardSize; i++) {
            if (board[row][i])
                return true;
        }

        // up left diagonal check
        while (rowT >= 0 && columnT >= 0) {
            if (board[rowT][columnT])
                return true;
            rowT--;
            columnT--;
        }
        rowT = row;
        columnT = column;

        // up right diagonal check
        while (rowT >= 0 && columnT < boardSize) {
            if (board[rowT][columnT])
                return true;
            rowT--;
            columnT++;
        }

        rowT = row;
        columnT = column;

        // down left diagonal check
        while (rowT < boardSize && columnT >= 0) {
            if (board[rowT][columnT])
                return true;
            rowT++;
            columnT--;
        }
        rowT = row;
        columnT = column;

        // down right diagonal check
        while (rowT < boardSize && columnT < boardSize) {
            if (board[rowT][columnT])
                return true;
            rowT++;
            columnT++;
        }
        return false;
    }
```

在展示核心代码之前,需要首先谈谈是如何来找地方来摆放棋子的.我使用的方法是以'列'为anchor,然后再在这一列的八行中分别尝试摆放棋子,如果满足条件,就在这个点放上棋子,如果不满足,行数加一继续往下走,如果遇到走不通的情况就回退一步.为了具体说明他是怎么回退的,我用一个具体的例子来阐述:
* 当摆放第一个棋子(row=0, column=0)的时候不会有冲突,所以直接在这里落子,现在棋盘的格局如下:

Q X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
 
* 当尝试摆放第二个棋子的时候,column到了第二列,所以它还是从第一行摆放,但是冲突了,所以他尝试第二行,但是还是冲突(斜线),所以他就开始尝试第三行,这里没问题,所以这一步之后,棋盘的格局如下:

Q X X X X X X X <br/>
X X X X X X X X <br/>
X Q X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>

* 当尝试摆放第三颗棋子的时候,第一到第四行都有冲突,所以只有放到第五行,棋盘的格局如下:

Q X X X X X X X <br/>
X X X X X X X X <br/>
X Q X X X X X X <br/>
X X X X X X X X <br/>
X X Q X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>

* 继续摆放第四颗棋子:

Q X X X X X X X <br/>
X X X Q X X X X <br/>
X Q X X X X X X <br/>
X X X X X X X X <br/>
X X Q X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>

* 继续摆放第五颗棋子:

Q X X X X X X X <br/>
X X X Q X X X X <br/>
X Q X X X X X X <br/>
X X X X Q X X X <br/>
X X Q X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>

* 继续摆放第六颗棋子,这个时候问题来了,因为第六列的所有行都无法摆放,所以这时就应该回退到上一列,让第五列的棋子继续尝试向下搜索(不能向上是因为在从上到下摆放的时候使用的贪心法,即遇到可以摆放的地方立马落子,所以上面的空间都是证明没用的),当找到一个可以落子的地方时就摆放棋子,然后重新尝试落第六个子:

Q X X X X X X X <br/>
X X X Q X X X X <br/>
X Q X X X X X X <br/>
X X X X Q X X X <br/>
X X Q X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>

* 

Q X X X X X X X <br/>
X X X Q X X X X <br/>
X Q X X X X X X <br/>
X X X X X X X X <br/>
X X Q X X X X X <br/>
X X X X X X X X <br/>
X X X X X X X X <br/>
X X X X Q X X X <br/>

* 我们发现即使第六个子变了位置之后,第七个子仍然不能下落,所以又回到第六列,这时第六列的所有可能性都已经用完了,所以然后回到第五列,这样不断递归下去直到从某一列到最后一列能都找到摆放的地方结束.

下面的代码是算法的核心,即怎样找到一种摆放的方法:

```java

   private boolean placeQueen_oneSolution(int column) {
        int row = 0;
        boolean success = false;

        while (row < boardSize) {
            if (column == boardSize) {
                printBoard();
                return true;
            }

            if (threaten(row, column)) {
                row++;
            } else {
                board[row][column] = true;
                success = placeQueen_oneSolution(column + 1);

                if (!success) {
                    board[row][column] = false;
                    row++;
                }
            }
        }
        return success;
    }
    
```

当上一次不成功时就返回false,同时把刚刚摆放的棋子拿掉(设为false),然后继续本次while循环,如果while循环完了都还没找到,就return false到上一次递归的地点. 执行上面的代码得到的格局如下:

Q X X X X X X X <br/>
X X X X X X Q X <br/>
X X X X Q X X X<br/>
X X X X X X X Q <br/>
X Q X X X X X X <br/>
X X X Q X X X X<br/>
X X X X X Q X X <br/>
X X Q X X X X X<br/>


这样就算找到了一条路径.但是如果我们想要找出所有的路径又该怎么办呢?其实很简单,只需要在找到一条路径后不停止搜索,从当前位置开始往下走,尝试去找其他的可能性,然后对之前的所有列都执行同样的步骤即可,方法如下:

```java

    private void placeQueen_allSolutions(int column) {
        int row = 0;

        while (row < boardSize) {
            if (column == boardSize) {
                printBoard();
                return;
            }

            if (threaten(row, column)) {
                row++;
            } else {
                board[row][column] = true;
                placeQueen_allSolutions(column + 1);

                board[row][column] = false;
                row++;
            }
        }
    }
    
```
最后发现有92种解法,与维基百科所属一致,ok收工.

最后贴出完整的代码:

```java

/**
 * Created by liqiushi on 3/20/14.
 */
public class Backtracking {
    int boardSize;
    boolean[][] board;

    int k = 1;

    public Backtracking(int boardSize) {
        this.boardSize = boardSize;
        board = new boolean[boardSize][boardSize];
    }

    public static void main(String[] args) {
        Backtracking backtracking = new Backtracking(8);
        backtracking.init();
        backtracking.placeQueen_oneSolution(0);
        backtracking.init();
        backtracking.placeQueen_allSolutions(0);
    }

    private void printBoard() {
        for (int i = 0; i < boardSize; i++) {
            for (int j = 0; j < boardSize; j++) {
                if (board[i][j]) {
                    System.out.print("Q ");
                } else {
                    System.out.print(". ");
                }
            }
            System.out.println();
        }
        System.out.println(k++);
    }

    private boolean placeQueen_oneSolution(int column) {
        int row = 0;
        boolean success = false;

        while (row < boardSize) {
            if (column == boardSize) {
                printBoard();
                return true;
            }

            if (threaten(row, column)) {
                row++;
            } else {
                board[row][column] = true;
                success = placeQueen_oneSolution(column + 1);

                if (!success) {
                    board[row][column] = false;
                    row++;
                }
            }
        }
        return success;
    }

    private void placeQueen_allSolutions(int column) {
        int row = 0;

        while (row < boardSize) {
            if (column == boardSize) {
                printBoard();
                return;
            }

            if (threaten(row, column)) {
                row++;
            } else {
                board[row][column] = true;
                placeQueen_allSolutions(column + 1);

                board[row][column] = false;
                row++;
            }
        }
    }

    private boolean threaten(int row, int column) {
        int rowT = row;
        int columnT = column;
        // same row check
        for (int i = 0; i < boardSize; i++) {
            if (board[row][i])
                return true;
        }

        // up left diagonal check
        while (rowT >= 0 && columnT >= 0) {
            if (board[rowT][columnT])
                return true;
            rowT--;
            columnT--;
        }
        rowT = row;
        columnT = column;

        // up right diagonal check
        while (rowT >= 0 && columnT < boardSize) {
            if (board[rowT][columnT])
                return true;
            rowT--;
            columnT++;
        }

        rowT = row;
        columnT = column;

        // down left diagonal check
        while (rowT < boardSize && columnT >= 0) {
            if (board[rowT][columnT])
                return true;
            rowT++;
            columnT--;
        }
        rowT = row;
        columnT = column;

        // down right diagonal check
        while (rowT < boardSize && columnT < boardSize) {
            if (board[rowT][columnT])
                return true;
            rowT++;
            columnT++;
        }
        return false;
    }

    private void init() {
        for (int i = 0; i < boardSize; i++) {
            for (int j = 0; j < boardSize; j++)
                board[i][j] = false;
        }
    }
}


```
