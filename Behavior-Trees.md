- [Introduction](#introduction)
- [Core Concepts](#core-concepts)
  * [Leaf Tasks](#leaf-tasks)
  * [Composite Tasks](#composite-tasks)
    - [Selector](#selector)
    - [Sequence](#sequence)
    - [Decorator](#decorator)
    - [Parallel](#parallel)
- [A Simple Example](#a-simple-example)
- [Behavior Tree API](#behavior-tree-api)

# Introduction #

For the last few years, behavior trees (BT) are the major formalism used in game industry to build complex AI behaviors. This success comes from the simplicity to understand, use and develop BT by non programmers. 

Behavior trees have been proposed as an improvement over Hierarchical State Machines for designing game AI of non-player characters (NPC). Their advantages over traditional AI approaches are being simple to design and implement, scalability when games get larger and more complex, and modularity to aid reusability and portability.

Behavior trees have been originally pioneered by [robotics](http://en.wikipedia.org/wiki/Robotics) and soon adopted for controlling AI behaviors in commercial games such as first-person-shooter [Halo 2](http://www.gamasutra.com/view/feature/130663/gdc_2005_proceeding_handling_.php) and life-simulation game [Spore](http://chrishecker.com/My_liner_notes_for_spore#Behavior_Tree_AI).

# Core Concepts #

The main problem with State Machines is the exponential growth of states and transitions. Even worse, states cannot be reused easily, without having to worry about transitions being invalid when they are reused for different portions of the AI logic. Essentially, State Machines lack a proper level of modularity.

Behavior Trees increase such modularity by encapsulating logic transparently within the states, making states nested within each other and thus forming a tree-like structure, and restricting transitions to only these nested states.
The root node branches down to more nodes until the leaf nodes are reached, and these leaf nodes are the base actions that define the behavior of the AI from a state beginning at the root. The concept of an AI state is now seen as a high level AI behavior, or task, whereby the links to nested children nodes define sub-tasks which make up the main behavior.
The leaf nodes are essentially then a group of basic actions that define a behavior.

The tasks of a behavior tree are formally constructed out of 2 classes of constructs: *leaf tasks* and *composite tasks*.
In the next sections, we cover these constructs, describing in detail their usage and applicabilities.

## Leaf Tasks ##

Leaf tasks are the terminal nodes of the tree and define low level actions which describe the overall behavior. They are typically implemented by user code, maybe in the form of a script, and can be something as simple as looking up the value of a variable in the game state, executing an animation, or playing a sound effect.

Leaf tasks consist of 2 types:
- **Actions** - which cause the execution of methods or functions on the game world, e.g. move a character, decrease health, etc...
- **Conditions** - which query the state of objects in the game world, e.g. location of character, amount of health, etc...

These actions and conditions form the definitions of behavior tasks. For example, the informal behavior tree below defines a task "Avoid Combat".
````
             +--------------+
             | Avoid combat |
             +--------------+
               /         \
+----------------+     +----------+
| Enemy visible? |     | Run avay |
+----------------+     +----------+
````
The root task is decomposed into two child tasks, the left child being a condition that an enemy is visible, and the right child being an action which makes the NPC run away.
Each of these actions and conditions can either succeed or fail, which in turn define whether the high-level behavioral task succeeds or fails.

## Composite Tasks ##
Composite tasks provide a standard way to describe relationships between child tasks, such as how and when they should be executed. Contrary to leaf tasks, which are defined by the user, composite nodes are predefined and provided by the behavior tree formalism. They allow you to build branches of the tree in order to organise their sub-tasks (the children). 
Basically, branches keep track of a collection of child tasks (conditions, actions, or other composites), and their behavior is based on the behavior of their children. Unlike actions and conditions, there are normally only a handful of composite tasks because with only a handful of different grouping behaviors we can build very sophisticated behaviors.
Especially, the composite tasks consist of *selectors*, *sequences*, *parallels* and *decorators* that will be described in the next subsessions.

### Selector ###
A selector runs each of its child behaviors in turn. It will return immediately with a success status code when one of its children runs successfully. As long as its children are failing, it will keep on trying. If it runs out of children completely, it will return a failure status code.

Selectors are used to choose the first of a set of possible actions that is successful.
For instance, a selector might represent a character wanting to reach safety. There may be multiple ways to do that: take cover, leave a dangerous area, and find backup. Such a selector will first try to take cover; if that fails, it will leave the area. If that succeeds, it will stop since there's no point also finding backup, as we've solved the character's goal of reaching safety. If we exhaust all options without success, then the selector itself has failed.

A selector node is graphically represented by a question mark.
![selector](https://cloud.githubusercontent.com/assets/2366334/4603480/5f2d3274-516d-11e4-80c5-8e5df55f9fd1.png)

### Sequence ###
A sequence runs each of its child behaviors in turn. It will return immediately with a failure status code when one of its children fails. As long as its children are succeeding, it will keep going. If it runs out of children, it will return in success. 

Sequences represent a series of tasks that need to be undertaken. Each of our reaching-safety actions in the slector example above may consist of a sequence. To find cover we'll need to choose a cover point, move to it, and, when we're in range, play a roll animation to arrive behind it. If any of the steps in the sequence fails, then the whole sequence has failed: if we can't reach our desired cover point, then we haven't reached safety. Only if all the tasks in the sequence are successful we can consider the sequence as a whole to be successful.

A sequence node is graphically represented by an arrow.
![sequence](https://cloud.githubusercontent.com/assets/2366334/4603496/61776ee0-516e-11e4-810a-cdc1a333d50d.png)

### Decorator ###
The name "decorator" is taken from object-oriented software engineering. The decorator pattern refers to a class that wraps another class, modifying its behavior. If the decorator has the same interface as the class it wraps, then the rest of the software doesn't need to know if it is dealing with the original class or the decorator.

In the context of a behavior tree, a decorator is a task that has one single child task and modifies its behavior in some way. You could think of it like a composite task with a single child. 
There are many different types of useful decorators:
- **AlwaysFail** will always fail no matter the wrapped task fails or succeeds.
- **AlwaysSucceed** will always succeed no matter the wrapped task succeeds or fails.
- **Include** grafts an external subtree. This decorator enhances behavior trees with modularity and reusability.
- **Invert** will succeed if the wrapped task fails and will fail if the wrapped task succeeds.
- **Limit** controls the maximum number of times a task can be run, which could be used to make sure that a character doesn't keep trying to barge through a door that the player has reinforced.
- **SemaphoreGuard** allows you to specify how many characters should be allowed to concurrently use the wrapped task. This decorator fails when it cannot acquire the semaphore. This allows a select task higher up the tree to find a different action that doesn't involve the contested resource. Imagine you have three monsters and you want one of them to chase the player and the rest to taunt the player.
- **UntilFail** will repeat the wrapped task until that task fails.
- **UntilSucceed** will repeat the wrapped task until that task succeeds.

There are many more decorators you might want to use, but I think these are enough for now.


### Parallel ###
T.B.D.

# A Simple Example #
T.B.D.

# Behavior Tree API #
T.B.D.

