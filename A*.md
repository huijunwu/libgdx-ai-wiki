- [Introduction](#introduction)
- [The Algorithm](#the-algorithm)
- [Performance & Complexity](#performance--complexity)
- [Demos and Further Reading](#demos-and-further-reading)


## Introduction ##
A* is a fundamental heuristic search algorithm for finding a path through a graph. Originally described in 1968, it has undergone many domain- and use-dependent improvements since its initial publication. Users unfamiliar with AI and specifically search-related terminology are encouraged to push through the technical-sounding definitions that are outlined next, as it is followed by a friendlier breakdown and discussion.

A* searches for a path over a discrete [state space](https://en.wikipedia.org/wiki/State_space). Hence, in many implementations the first step is to apply a coarse discretization over some continuous domain. For example, by applying a grid, [Delaunay triangulation](https://en.wikipedia.org/wiki/Delaunay_triangulation), or other quickly-computed (i.e. [polynomial time](http://mathworld.wolfram.com/PolynomialTime.html)) abstraction over the state space. 

![A-star search](http://www.entangledloops.com/img/a-star-search-1.png)

In a game, the discrete graph is almost always a pre-sized grid applied over a 2D or 3D map. The search is performed on the grid by selecting a grid node closest to the agent (PC or NPC) and then progressively searching through the graph for a node closest to some goal location (for example, where the user has clicked). The grid may initially be aligned to the agent, the goal, or neither for performance reasons, such as in repeated search. Several techniques exist to remediate the situation where a path cannot be found through the graph _even though a path exists_ in the complete continuous space. Common techniques employ refining or relocating the graph's position.

A* is **not** inherently a multi-agent path-finding algorithm. Multi-agent path-planning is a separate, more complex topic that requires discussion of stochastic processes and reliability of the available information. Other algorithms and adaptations of A* exist specifically for handling those situations.

A* is only guaranteed to find the _shortest possible path_ in a graph if two conditions are unquestionably met by the heuristic function:
- **Consistency** - A consistent (**monotone**) heuristic never violates the [triangle inequality](https://en.wikipedia.org/wiki/Triangle_inequality). In laymen's terms, this means it never returns mutually incompatible distance estimates from two different nodes.
- **Admissibility** - An admissible heuristic never overestimates the distance to the goal. It is both **complete** and **optimal**. Respectively, these mean that
    - if a path exists, it will be found, and
    - it will be the shortest path.

## The Algorithm ##
### Description ###

#### Definitions ####
This description will use common game terminology to reduce the amount of prerequisite knowledge or further reading required to fully grasp the algorithm and its readily accessible variants.

A **node** in the graph represents a possible map location. In this discussion, we will imagine a node `n` as a single point with an (x, y) coordinate. However, it should be noted that a node can be any abstract notion, so long as a distance-to-goal function can be defined for it. For example, A* can be used for DNA sequence alignment, where the distance function `h(n)` represents the number of mismatched base pairs in the alignment. A perfect alignment would have 0 mismatched base pairs, and thus have a "distance" of 0 to the goal. 

The output of A* is either nil (the empty set), or a **path**--an ordered set of nodes. The input is a start node, a **goal test** function, an **expansion** function, and a **heuristic** function.

The **open list** or simply "open", is the list of all available un-expanded nodes. It is usually implemented as a priority queue.

The **closed list** is an optional record of all expanded nodes. It is used to prevent repeated search and infinite loops.

#### Heuristic Function `h(n)` ####
The purpose of the heuristic function is to return an estimate of the distance from a node `n` to the goal node.  It is not expected to be (and shouldn't be) a perfect heuristic, meaning a function that knows the precise distance from any location to the goal. If a perfect heuristic were used--or for that matter, even existed--then the problem of search would be solved outright! That is, if you have a magic function that always knows the correct, true distance through the graph from any point, then there is no reason to search for a path in the first place. In that case, you could simply reverse engineer such a function and use it directly without needing A*. However, no such general heuristic is known, nor is likely to ever be known for all domains. The alternative is using a distance-estimator `h(n)`, and then making the best local decision possible at each point of the search.

The exact heuristic function used will determine the output and could also affect the time complexity.
For example, if an inconsistent `h(n)` is chosen, the path returned is not guaranteed to be optimal. If a complex `h(n)` is used, then the time consumed at each call might bring A* to a grinding halt. The idea is to choose `h(n)` so that it is well-balanced and reflects the domain, while not draining too heavily on available resources. If is also possible to use multiple heuristics, although doing so effectively will not be discussed here.

##### Examples #####

Some standard domain-independent heuristic functions include:
- [Manhattan distance](https://en.wiktionary.org/wiki/Manhattan_distance) from node to goal (ignoring obstacles)
- [Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance) (generalization of Pythagorean)
- [Chebyshev distance](https://en.wikipedia.org/wiki/Chebyshev_distance) (max of vector components)
- Octile distance (allows diagonal grid movement)

#### Mechanics ####
A* works in the following manner:
- Sort open on `n.f`
- Removing the first node `n` from open (lowest `n.f`, closest to goal)
- Expand `n` by generating its children (usually proximal neighbors in the search graph)
- Goal check
- Add `n` to closed
- Compute `f(n) = g(n) + h(n)` for each child node
- Add each child to open

So far, the only mysterious function is `g(n)`, which is simply the **cost** of reaching `n`. The cost is usually an integer value that is simply incremented once for each step taken since departing the start node. It is meant to be a measurement that takes into account the work done by the agent in traversing the solution. 

Contrast this with a greedy approach, which would only look at `h(n)` before selecting a child node to expand (best-first-search or BFS). Such behavior can lead to poor performance, poor plans, or both in certain situations. For example, consider the following pathological map: 

![A-star search](http://www.entangledloops.com/img/greedy-solution-1.png)

Here the dark green tile represents the start, red is the goal, grey are walls, blue are expanded nodes, and light green are nodes still on open. In the bottom left-hand corner some statistics can be seen. Notice that the this search returns the suboptimal path, and also does so in more time than an A* implementation that considers `g(n)`, which--on the same map--found the optimal solution of length 28.41 in 4.0000 ms.

On the other hand, a greedy search may find a solution when A* simply cannot due to time or memory bounds. A greedy solution may also be optimal in certain circumstances, such as the [knapsack problem](https://en.wikipedia.org/wiki/Knapsack_problem) when real-valued weights are used (for this reason, integer value weights are used in the canonical problem definition). The point is that you should choose the search strategy that best fits your domain; you shouldn't necessarily defer to the worst-case complexity bounds of an algorithm when selecting your approach.

### Pseudocode ###
![A* Pseudocode](http://www.entangledloops.com/img/a-star-pseudocode.png)

Note that the _makePath_ function simply reconstructs the path from goal to the start by following parent pointers backwards, then reversing the list.

## Performance & Complexity ##

In the worst case, A* devolves into a [breadth-first search](https://en.wikipedia.org/wiki/Breadth-first_search) (BFS) with complexity O(b^d), where b is the branching factor and d is the depth of the goal.

The simplistic pseudocode above does not include explicit reference to many performance-enhancing possibilities. For example, closed should be tracked as a hash table and search could be performed in parallel (advanced, see [further reading](#demos-and-further-reading)). Overall, the performance of A* depends heavily on
- how and how often sorting is performed,
- the underlying data structures used,
- complexity of the heuristic function(s),
- whether duplicate detection is performed, and
- whether optimality is required.

Also notice that A* must perform an exhaustive search and obtain a result before any plan is returned. This is because vanilla A* is not an **[anytime algorithm](https://en.wikipedia.org/wiki/Anytime_algorithm)** and therefore cannot return partial plans--a significant weakness for most games. An anytime adaptation can be found [here](http://papers.nips.cc/paper/2382-ara-anytime-a-with-provable-bounds-on-sub-optimality.pdf). A **realtime adaptive** improvement can be found [here](https://www.cs.cmu.edu/~motionplanning/papers/sbp_papers/integrated2/koenig_realtime_adaptive_astar_aamas06.pdf). Hopefully implementations of these algorithms will be added to gdxAI in the near future.

## Demos and Further Reading ##

Sorting is critical to the performance and behavior of A*. Note that [tie-breaking](http://movingai.com/astar.html) should be done on `h`, since it more accurately captures the available goal information.

[Weighted A*](https://www.cs.cmu.edu/~motionplanning/lecture/Asearch_v8.pdf) is a simple approach to improve performance when optimality is not required, like in most games. It requires you to multiply the value of `h(n)` by some constant, which you should determine by experimenting.

An interactive javascript demo (with some minor bugs) can be found [here](https://qiao.github.io/PathFinding.js/visual).

A video that shows all expanded nodes in white can be seen [here](https://www.youtube.com/watch?v=19h1g22hby8). Another can be seen [here](https://www.youtube.com/watch?v=J-ilgA_XNI0).

The fastest known implementation was written in C++ by Google engineer [Ethan Burns](https://eatoasts.appspot.com). His site contains links to a [codebase](https://github.com/eaburns/search) with more advanced implementations, including parallelization.

You can find other variants of A* [here](http://theory.stanford.edu/~amitp/GameProgramming/Variations.html).