To build a general-purpose AI system, we need to have some infrastructure that makes it easy to get the right information to the right bits of AI code at the right time. Communication is the key. It is essential for even simple AI, but comes into its own when multiple characters need to coordinate their behaviors.

In this section we'll look at one common approach for getting information to and between characters in a game: [[Message Handling]].

[Polling](http://en.wikipedia.org/wiki/Polling_%28computer_science%29) is another common approach, but currently it's not supported by the framework. Also, some developers find that it is easier to manage the game information only using events (messages).

