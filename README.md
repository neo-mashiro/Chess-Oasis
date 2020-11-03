# Chess-Oasis

A chess library in 300 lines of code, architectured in a way such that it is independent of the graphical user interface with low coupling. The library also includes an intermediate level Minimax AI bot.

This module is a tiny component of the [Mana-Oasis](https://github.com/neo-mashiro/Mana-Oasis) project. It only focuses on the game logic while UI elements are handled elsewhere on caller's side. For this reason, the module can be used by any console/web application or game engine provided that the caller does not misbehave. That said, you may need to run unit tests or handle logging/exceptions on the fly, in that case, simply modify `Debug.Log()` (for Unity) to the equivalent statement of your application such as:

```c#
Console.WriteLine("Your message");
MessageBox.Show("Your message");

Debug.WriteLine("Your message");
Trace.WriteLine("Your message");

UE_LOG(LogTemp, Warning, TEXT("Your message"));
GEngine->AddOnScreenDebugMessage(-1, 15.0f, FColor::Red, "Your message");
```

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




# Reference

https://science.sciencemag.org/content/317/5844/1518
