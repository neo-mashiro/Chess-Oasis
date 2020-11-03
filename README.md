# Chess-Oasis

A chess library in 300 lines of code, architectured in a way such that it is independent of the graphical user interface with low coupling. The library also includes an intermediate level Minimax AI bot.

This module is a tiny component of the [Mana-Oasis](https://github.com/neo-mashiro/Mana-Oasis) project. It only focuses on the game logic while UI elements are handled elsewhere on caller's side. For this reason, the module can be used by any console application or game engine. Here is an example of how to call it in Unity3D's Monobehavior script:

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




# Reference

https://science.sciencemag.org/content/317/5844/1518
