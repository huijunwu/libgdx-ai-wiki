## Introduction ##
Hierarchical pathfinding plans a route in much the same way as people would. You plan an overview route first and then refine it as needed. The high-level overview route to get to the Rome office from the Paris office might be "Go to the airport, catch a flight, and get a cab from the airport." 

Each stage of the path will consist of another route plan. To get to the airport, for example, you need to know the route. The first stage of this route might be to get to the car. This, in turn, might require a plan to get to the rear parking lot, which in turn will require a plan to maneuver around the desks and get out of the office.

This is a very efficient way of pathfinding. To start with, we plan the abstract route, take the first step of that plan, find a route to complete it, and so on down to the level where we can actually move. After the initial multi-level planning, we only need to plan the next part of the route when we complete a previous section.

The plan at each level is typically simple, and we split the pathfinding problem over a long period of time, only doing the next bit when the current bit is complete.

As we'll see, with hierarchical pathfinding you gain performance at the possible cost of path optimality. However, in games, missing the optimal path is not a real problem most of the times, as long as the calculated path looks reasonable.

## The API ##
### Hierarchical Path Finder ###
A [HierarchicalPathFinder](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/HierarchicalPathFinder.html) can find a path in an arbitrary [HierarchicalGraph](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html) using any [PathFinder](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/PathFinder.html), known as level path finder, on each level of the hierarchy.

Pathfinding on a hierarchical graph applies the level path finder algorithm several times, starting at a high level of the hierarchy and working down. The results at high levels are used to limit the work it needs to do at lower levels.

### Hierarchical Graph ###
The `HierarchicalGraph` is a multilevel graph that can be traversed by a `HierarchicalPathFinder` at any level of its hierarchy. Especially, the hierarchical path finder calls the [HierarchicalGraph.setLevel()](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html#setLevel-int-) method to switch the graph into a particular level. All future calls to the [HierarchicalGraph.getConnections()](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html#getConnections-Object-) method of the hierarchical graph then act as if the graph was just a simple, non-hierarchical graph at that level. This way, the level path finder has no way of telling that it is working with a hierarchical graph and it doesn't need to, meaning that for the level path finder you can use any path finder implementation, such as the regular [IndexedAStarPathFinder](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/indexed/IndexedAStarPathFinder.html), for instance.

### Nodes ###
Nodes in a hierarchical graph form clusters by grouping locations together. The individual locations for a whole
room, for example, can be grouped together. There may be any number of navigation points in the room, but for higher level plans they can be treated as one. This group can be treated as a single node in the pathfinder.

{--Picture needed--}

This process can be repeated as many times as needed. The nodes for all the rooms in one building can be combined into a single group, which can then be combined with all the buildings in a complex, and so on. The final product is a hierarchical graph. At each level of the hierarchy, the graph acts just like any other graph you might pathfind on.

To allow pathfinding on this graph, you need to be able to convert a node between different levels of the hierarchy. This is done by the [HierarchicalGraph.convertNodeBetweenLevels()](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html#convertNodeBetweenLevels-int-Object-int-) method. There are two cases:
- **Increasing the level of a node:** The conversion of a node at the lowest level of the graph (which is derived from the character's position in the game level) to one at a higher level is the equivalent of the **quantization** step in regular graphs. A typical implementation will store a mapping from nodes at one level to groups at a higher level. Note that these are many-to-one relationships.
- **Decreasing the level of a node:** When converting a node at a higher level to one at a lower level, one node might map to any number of nodes at the next level down. This one-to-many relationship is just the same process as localizing a node into a position for the game. There are any number of positions in the node, but we select one in **localization**. The same thing needs to happen in the `convertNodeBetweenLevels` method. You need to select a single node that can be representative of the higher level node. This is usually a node near the center, or it could be the node that covers the greatest area or the most connected node (an indicator that it is a significant one for route
planning).


### Connections ###
T.B.D


