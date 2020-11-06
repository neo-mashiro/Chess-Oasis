# Chess-Oasis

A chess library in 300 lines of code, architectured in a way such that it is independent of the graphical user interface with low coupling. The library also includes an intermediate level Minimax AI bot.

This module is a tiny component of the [Mana-Oasis](https://github.com/neo-mashiro/Mana-Oasis) project. It only focuses on the game logic while UI elements are handled elsewhere on caller's side. For this reason, the module can be used by any console/web application or game engine provided that the caller does not misbehave. That said, you may want to run some unit tests, print logging messages or handle exceptions on the fly, in that case, simply modify the Unity function calls `Debug.Log()`, `Debug.LogException()` or alike to the equivalents of your application:

```c#
Console.WriteLine("Your message");
MessageBox.Show("Your message");

Debug.WriteLine("Your message");
Trace.WriteLine("Your message");

UE_LOG(LogTemp, Warning, TEXT("Your message"));
UE_LOG(LogHAL, Fatal, TEXT("Your message"), ...);

GEngine->AddOnScreenDebugMessage(-1, 15.0f, FColor::Red, "Your message");
...
```

## API

Ranks and files are numbered from 1 to 8, you can access a cell on board either by a `rank, file` pair or a string such as `4B`. To see the basic rules of chess, check [Wikipedia](https://en.wikipedia.org/wiki/Rules_of_chess).

**Note**: for simplicity, _En passant_ and _Castling_ movements are not available. Besides, _checkmate_ is not auto detected as in most 2D chess games, since the UI side may want to give players more freedom or need to play a death animation before the game ends.

```c#
ChessBoard board = new ChessBoard();

bool result = board.GameOver;  // check if game is over
int count = board.TotalMoves;  // total number of moves

// index by rank, file
var cell = board[7, 6];
var cell = board[9, 1];  // IndexOutOfRangeException

// index by string
var cell = board["1D"];
var cell = board["0X"];  // IndexOutOfRangeException

// enumerator
foreach (var piece in board) {
    if (piece != null) {
        // you should not need to access the properties of a piece (but you can)
        Console.Writeline(p);
    }
}

// retrieve all legal moves a cell can make
List<string> moves = board.GetLegalMoves("5F");

// retrieve the next move made by AI
(int rank, int file) move = board.MinimaxAI();

// make a move (you should first check if it's legal)
board.Move("2C", "4C");
board.Move("3E", "4B");  // if "3E" is empty, do nothing
board.Move("1H", "3X");  // IndexOutOfRangeException

// the white or black player decides to surrender
board.Resign("White");

// reset the chessboard to initial state
board.Reset();
```

## Example in Unity

Here is an example of how to use the library in Unity3D's Monobehavior script:

```c#
public void Start() {
    Chessboard board = new Chessboard();
    ....
}

public void Update() {
    if () {
        ....
    }
}
```

## Reference

- Schaeffer, Jonathan & Björnsson, Yngvi & Kishimoto, Akihiro & Müller, Martin & Lake, Robert & Lu, Paul & Sutphen, Steve. (2007). Checkers Is Solved. Science. 317. 1518-1522. 10.1126/science.1144079. https://science.sciencemag.org/content/317/5844/1518
