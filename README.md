# Chess-Oasis

A chess library in 300 lines of code, architectured in a way such that it is independent of the graphical user interface with low coupling. The library also includes an intermediate level Minimax AI bot.

This module is a tiny component of the [Mana-Oasis](https://github.com/neo-mashiro/Mana-Oasis) project. It only focuses on the game logic while UI elements are handled elsewhere on caller's side. For this reason, the module can be used by any console/web application or game engine provided that the caller does not misbehave. That said, you may want to run some unit tests, print logging messages or handle exceptions on the fly, in that case, simply replace the Unity function calls `Debug.Log()`, `Debug.LogException()` or alike to the equivalents of your application:

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
string winner = board.Winner;  // winner
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

// convert between a (rank, file) pair and its string position
string cell = Map.ToStrPosition(6, 8);
string cell = Map.ToStrPosition(9, 1);  // IndexOutOfRangeException

(int rank, int file) cell = Map.ToIntPosition("6H");
(int rank, int file) cell = Map.ToIntPosition("0A");  // IndexOutOfRangeException
```

## Example in Unity

Here is an example (not tested) of how to use the library in Unity3D's Monobehavior script.

![board](chess/ChessBoard.png)

Assume that you have a 3D chessboard consists of 64 cube cells, each has a volumetric light child object set to inactive on game start. Aside from the board, each player is in control of 16 piece objects. On each frame, once the player left clicks on a cell, all cells that he can legally move to are spotlighted, the player then simply right click on a reachable cell to make a move, followed by the Minimax AI to automatically make another move, and so forth...

In concrete, the steps to make a move may look like this:
1. make sure spotlights are turned off
2. the API `Move()`, `MinimaxAI()` is called to update the underlying board data
3. start a coroutine, navigate the selected piece (NavMeshAgent) to the destination and wait
4. on completion, if the target cell is occupied, play fight animation to kill the enemy, set it to inactive
5. check if game is over, if so, congratulate the winner and open an UI dialog...

```c#
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using UnityEngine.AI;
using UnityExtensions;

namespace Chess {
    // to be attached on the chessboard game object
    public class ChessController : MonoBehaviour {
        private Camera _camera;
        private ChessBoard _board;
        private Transform[] _cells;
        private Transform[] _pieces;
        private Transform[] _spotlights;

        private RaycastHit _hit;
        private bool _suspendUpdate;

        private Transform _currentCell;
        private static readonly int Fight = Animator.StringToHash("Fight");

        private const int LayerMask = 1 << 8;

        private void Start() {
            _camera = Camera.main;
            _board = new ChessBoard();
            _cells = transform.GetAllDirectChildren();
            _pieces = GameObject.FindGameObjectsWithTag("Piece").Select(go => go.transform).ToArray();
            _spotlights = _cells.Select(t => t.Find("Volumetric Light")).ToArray();
        }

        private void Update() {
            if (_suspendUpdate) { return; }

            if (Input.GetMouseButtonDown(0)) {
                var ray = _camera.ScreenPointToRay(Input.mousePosition);

                if (Physics.Raycast(ray, out _hit, 100.0f, LayerMask)) {
                    _currentCell = _hit.transform;
                    var cells = _board.GetLegalMoves(_currentCell.name);
                    SpotlightOn(cells);
                }
                else {
                    _currentCell = null;
                    SpotlightOff();
                }
            }

            else if (Input.GetMouseButtonDown(1)) {
                var ray = _camera.ScreenPointToRay(Input.mousePosition);

                if (Physics.Raycast(ray, out _hit, 100.0f, LayerMask)) {
                    if (_hit.transform.GetChild(0).gameObject.activeSelf) {
                        SpotlightOff();
                        Move(_currentCell, _hit.transform);
                    }
                }
            }
        }

        private IEnumerator AnimateMove(Transform from, Transform to) {
            var sourcePiece = GetPieceOnCell(from);
            var targetPiece = GetPieceOnCell(to);

            var agent = sourcePiece.GetComponent<NavMeshAgent>();
            agent.SetDestination(to.position);

            yield return new WaitForSeconds(5);

            if (agent.pathStatus == NavMeshPathStatus.PathComplete) {
                if (!ReferenceEquals(targetPiece, null)) {
                    sourcePiece.GetComponent<Animator>().SetTrigger(Fight);
                    yield return new WaitForSeconds(2);
                    targetPiece.gameObject.SetActive(false);
                }
            }

            CheckGameOver();
        }

        private void Move(Transform from, Transform to) {
            _suspendUpdate = true;

            _board.Move(from.name, to.name);
            StartCoroutine(AnimateMove(from, to));
            _currentCell = null;

            var (source, target) = _board.MinimaxAI();
            StartCoroutine(AnimateMove(transform.Find(source), transform.Find(target)));

            _suspendUpdate = false;
        }

        private void CheckGameOver() {
            if (_board.GameOver) {
                Debug.Log($"Game over, {_board.Winner} wins!");
            }
        }

        private void SpotlightOn(List<string> cells) {
            for (var i = 0; i < _cells.Length; i++) {
                var on = cells.Contains(_cells[i].name);
                _spotlights[i].gameObject.SetActive(on);
            }
        }

        private void SpotlightOff() {
            foreach (var spotlight in _spotlights) {
                spotlight.gameObject.SetActive(false);
            }
        }

        private Transform GetPieceOnCell(Transform cell) {
            return _pieces.FirstOrDefault(piece =>
                piece.position.x > cell.position.x && piece.position.x < cell.position.x + 1 &&
                piece.position.z > cell.position.z && piece.position.z < cell.position.z + 1);
        }
    }
}

```

## Reference

- Schaeffer, Jonathan & Björnsson, Yngvi & Kishimoto, Akihiro & Müller, Martin & Lake, Robert & Lu, Paul & Sutphen, Steve. (2007). Checkers Is Solved. Science. 317. 1518-1522. 10.1126/science.1144079. https://science.sciencemag.org/content/317/5844/1518
