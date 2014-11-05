## Introduction ##
Hierarchical pathfinding plans a route in much the same way as people would. You plan an overview route first and then refine it as needed. The high-level overview route to get to the Rome office from the Paris office might be "Go to the airport, catch a flight, and get a cab from the airport." 

Each stage of the path will consist of another route plan. To get to the airport, for example, you need to know the route. The first stage of this route might be to get to the car. This, in turn, might require a plan to get to the rear parking lot, which in turn will require a plan to maneuver around the desks and get out of the office.

This is a very efficient way of pathfinding. To start with, we plan the abstract route, take the first step of that plan, find a route to complete it, and so on down to the level where we can actually move. After the initial multi-level planning, we only need to plan the next part of the route when we complete a previous section.

The plan at each level is typically simple, and we split the pathfinding problem over a long period of time, only doing the next bit when the current bit is complete.

As we'll see, with hierarchical pathfinding you gain performance at the possible cost of path optimality. However, in games, missing the optimal path is not a real problem most of the times, as long as the calculated path looks reasonable.

