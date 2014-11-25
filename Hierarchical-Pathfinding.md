## Introduction ##
Hierarchical pathfinding plans a route in much the same way as people would. You plan an overview route first and then refine it as needed. The high-level overview route to get to the Rome office from the Paris office might be "Go to the airport, catch a flight, and get a cab from the airport." 

Each stage of the path will consist of another route plan. To get to the airport, for example, you need to know the route. The first stage of this route might be to get to the car. This, in turn, might require a plan to get to the rear parking lot, which in turn will require a plan to maneuver around the desks and get out of the office.

This is a very efficient way of pathfinding. To start with, we plan the abstract route, take the first step of that plan, find a route to complete it, and so on down to the level where we can actually move. After the initial multi-level planning, we only need to plan the next part of the route when we complete a previous section.

The plan at each level is typically simple, and we split the pathfinding problem over a long period of time, only doing the next bit when the current bit is complete.

Also, for many real-time pathfinding applications, the complete path is not needed. Knowing the first few moves of a valid path often suffices, allowing an agent to start moving in the right direction. Subsequent events may result in the agent having to change its plan, obviating the need for the rest of the path. The obvious advantage here is that no effort has been wasted on computing a path to a goal node that was never needed.

As we'll see, with hierarchical pathfinding you gain performance at the possible cost of path optimality. However, in games, missing the optimal path is not a real problem most of the times, as long as the calculated path looks reasonable.

## The API ##
### Hierarchical Path Finder ###
A [HierarchicalPathFinder](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/HierarchicalPathFinder.html) can find a path in an arbitrary [HierarchicalGraph](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html) using any [PathFinder](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/PathFinder.html), known as level path finder, on each level of the hierarchy.

Pathfinding on a hierarchical graph applies the level path finder algorithm several times, starting at a high level of the hierarchy and working down. The results at high levels are used to limit the work it needs to do at lower levels.

On each run of the algorithm, the process of lowering the level and resetting the end location to the representative node of the closest group is repeated until the lowest level of the graph is reached. This way you get a plan in detail for what the character needs to do immediately. Note that initially you don't need to know the fine detail up to the end of the plan, because you'll work that out nearer the time. And you can be confident that, even though the algorithm has only looked in detail at the first few steps, making those initial moves will still lead the character to his goal in a sensible way. This is exactly the basic idea behind hierarchical pathfinding.

### Hierarchical Graph ###
The `HierarchicalGraph` is a multilevel graph that can be traversed by a `HierarchicalPathFinder` at any level of its hierarchy. Especially, the hierarchical path finder calls the [HierarchicalGraph.setLevel()](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html#setLevel-int-) method to switch the graph into a particular level. All future calls to the [HierarchicalGraph.getConnections()](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html#getConnections-Object-) method of the hierarchical graph then act as if the graph was just a simple, non-hierarchical graph at that level. This way, the level path finder has no way of telling that it is working with a hierarchical graph and it doesn't need to, meaning that for the level path finder you can use any path finder implementation, such as the regular [IndexedAStarPathFinder](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/indexed/IndexedAStarPathFinder.html), for instance.

### Nodes ###
Nodes in a hierarchical graph form clusters by grouping locations together. The individual locations for a whole
room, for example, can be grouped together. There may be any number of navigation points in the room, but for higher level plans they can be treated as one. This group can be treated as a single node in the pathfinder.

{--Picture needed--}

This process can be repeated as many times as needed. The nodes for all the rooms in one building can be combined into a single group, which can then be combined with all the buildings in a complex, and so on. The final product is a hierarchical graph. At each level of the hierarchy, the graph acts just like any other graph you might pathfind on.

To allow pathfinding on this graph, you need to be able to convert a node between different levels of the hierarchy. This is done by the [HierarchicalGraph.convertNodeBetweenLevels()](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/pfa/HierarchicalGraph.html#convertNodeBetweenLevels-int-N-int-) method. There are two cases:
- **Increasing the level of a node:** The conversion of a node at the lowest level of the graph (which is derived from the character's position in the game level) to one at a higher level is the equivalent of the **quantization** step in regular graphs. A typical implementation will store a mapping from nodes at one level to groups at a higher level. Note that these are many-to-one relationships.
- **Decreasing the level of a node:** When converting a node at a higher level to one at a lower level, one node might map to any number of nodes at the next level down. This one-to-many relationship is just the same process as localizing a node into a position for the game. There are any number of positions in the node, but we select one in **localization**. The same thing needs to happen in the `convertNodeBetweenLevels` method. You need to select a single node that can be representative of the higher level node. This is usually a node near the center, or it could be the node that covers the greatest area or the most connected node (an indicator that it is a significant one for route
planning). The [HierarchicalTiledAStarTest](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/pfa/tests/HierarchicalTiledAStarTest.java) uses a fixed node at a lower level, generated by finding the node of type floor which is closest to the center of all those mapped to the same higher level node, see the `findFloorTileClosestToCenterOfMass()` method in [HierarchicalTiledGraph.java](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/pfa/tests/tiled/hrchy/HierarchicalTiledGraph.java). This is a simple geometric preprocessing step that doesn't require human intervention. This node is then stored with the higher level node during graph creation, and it can be returned when needed without additional processing.

### Connections ###
The connections between higher level
nodes need to reflect the ability to move between grouped areas. If any low-level node in one group is connected to any low-level node in another group, then a character can move between the groups, and the two groups should have a connection.

The **cost of a connection** between two groups should reflect the difficulty of traveling between them. This can be specified manually, or it can be calculated from the cost of the low-level connections between those groups.
````
                   +--+--+      
                   |A1|C1|      
                   +--+--+      
                      |C2|      
    A                 +--+      
     \                |C3|      
      C-D             +--+--+--+
     /                |C4|C5|D1|
    B                 +--+--+--+
                   +--+--+--+--+
                   |B1|C6|C7|D2|
                   +--+--+--+--+
                                
   Level 1           Level 0   
````
The figure below shows that the cost of moving from group C to group D depends on whether you entered group C from group A (a cost of 5) or from group B (a cost of 2). In general, the grouping should be chosen to minimize this problem, but it cannot be resolved easily.

For the sake of simplicity, the `HierarchicalTiledAStarTest` uses a cost of 1 for all the connections between groups.
However, there are three heuristics that are commonly used, straight or blended, to calculate the connection
cost between groups:
- **Minimum Distance:** This heuristic says that the cost of moving between two groups is the cost of the cheapest link between any nodes in those groups. This makes sense because the pathfinder will try to find the shortest route between two locations. In the example above, the cost of moving from C to D would be 1. Note that if you entered C from either A or B, it would take more than one move to get to D. The value of 1 is almost certainly too low. Actually, this heuristic assumes that there will never be any cost to moving around the nodes within a group, thus it's very optimistic.
- **Maximin Distance:** For each incoming link, the minimum distance to any suitable outgoing link is calculated. This calculation is usually done with a pathfinder. The largest of these values is then added to the cost of the outgoing link and used as the cost between groups. In the example, to calculate the cost of moving from C to D, two costs are calculated: the minimum cost from C1 to C5 (4) and the minimum cost from C6 to C7 (1). The largest of these (C1 to C5) is then added to the cost of moving from C5 to D1 (1). This leaves a final cost from C to D of 5. To get from C to D from anywhere other than C1, this value will be too high. Actually, this heuristic finds one of the largest possible costs and always uses that, thus it's rather pessimistic.
- **Average Minimum Distance:** This heuristic is calculated in the same way as the maximin distance, but the values are averaged, rather than simply selecting the largest. In our example, to get from C to D coming from B (i.e., via C6 and C7), the cost is 2, and when coming from A (via C2 to C5) it is 5. So the average cost of moving from C to D is 3.5. Actually, this heuristic gives the average cost you'll pay over lots of different pathfinding requests, thus it's pragmatic and can be a good general choice.

