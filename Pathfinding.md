Game characters usually need to move around their level. Often, they know which is the place they want to go, but they don't know in advance where they'll need to move to finally reach their goal.

For each of these characters the AI must be able to calculate a suitable route through the game level to get from where it is now to its goal. We'd like the route to be sensible and as short or rapid as possible. It doesn't look smart if your character walks from the kitchen to the lounge via the attic, right?

This is pathfinding, sometimes called path planning, and it is everywhere in game AI. Pathfinding sits on the border between [[Decision Making]] and [[Movement AI]]. Typically, the goal is decided by some decision making technique, then the pathfinder works out how to get there and finally the movement control system moves the character along the path.

In this section we'll look at 
- [[Pathfinding API]]
- [[A*: A Little Bit of Theory|A*]]
- [[Indexed A*]]
- [[Hierarchical Pathfinding]]
- [[Path Smoothing]]