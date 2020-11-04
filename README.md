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

- Schaeffer, Jonathan & Björnsson, Yngvi & Kishimoto, Akihiro & Müller, Martin & Lake, Robert & Lu, Paul & Sutphen, Steve. (2007). Checkers Is Solved. Science. 317. 1518-1522. 10.1126/science.1144079. https://science.sciencemag.org/content/317/5844/1518
