### Graph Representation ###

We need to represent our graph in such a way that pathfinding algorithms such as A* and Dijkstra can work with it.
As we will see, the algorithms need to find out the outgoing connections from any given node.
And for each such connection, they need to have access to its cost and destination.

The API defines the following interfaces to represent a graph for pathfinding:

* The [Graph](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/Graph.html) interface simply returns an array of connection objects for any node that is queried.
* The [Connection](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/Connection.html) interface defines methods to retrieve the source node, the destination node and the cost.

A simple implementation of the `Graph` interface would store the connections for each node and simply return the list. Each `Connection` would have the cost and end node stored in memory.

A more complex implementation might calculate the cost only when it is required, using information from the current structure of the game level.

The [DefaultConnection](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/DefaultConnection.html) class implements a very simple connection with constant cost of 1.

Notice that there is no specific data type for a node in this API, because we don't need to specify one. We simply use generics to represent a node, which makes the API more flexible. In many cases it is sufficient just to give nodes a unique number (we will see that this is a particularly powerful implementation because it opens up some specific, fast optimizations of the A* algorithm, see [[Indexed A*]]. In other situations, for example, nodes can be as complex as an entire subgraph, see the [[Hierarchical Pathfinding]] section.


### Finding a Path ###

To let you find a path in a graph the API provides the following interfaces:

* The [GraphPath](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/GraphPath.html) interface represents a path in a [Graph](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/Graph.html). Notice that a path can be defined in terms of nodes or connections. When you use a path made up of connections multiple edges between the same pair of nodes can be discriminated. Also, the `GraphPath` interface extends [java.lang.Iterable](http://docs.oracle.com/javase/7/docs/api/java/lang/Iterable.html), meaning that a path can be the target of the "foreach" statement.

* The [Heuristic](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/Heuristic.html) interface represents a function that generates estimates of the cost to move from a given node to the goal. With a heuristic function pathfinding algorithms can choose the node that is most likely to lead to the optimal path. The notion of "most likely" is controlled by the heuristic itself. If the heuristic is accurate, then the algorithm will be efficient. If the heuristic is terrible, then it can perform even worse than other algorithms that don't use any heuristic function such as Dijkstra.

* The [PathFinder](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/PathFinder.html) interface define methods to find a `GraphPath` from a start node to a goal node in an arbitrary `Graph`. A fully implemented path finder can perform both interruptible (time slicing over multiple frames) and non-interruptible (all in one frame) searches. If a specific path finder is not able to perform one of the two type of search then the corresponding method should throw an `UnsupportedOperationException`. See the [[Scheduling API|Scheduling]] for further information about interruptible tasks.



