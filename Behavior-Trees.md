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
  * [Task Class Hierarchy](#task-class-hierarchy)
  * [Using Data for Inter-Task Communication](#using-data-for-inter-task-communication)
  * [Text Format](#text-format)
  * [Task Attributes and Constraints](#task-attributes-and-constraints)
  * [Behavior Tree Libraries](#behavior-tree-libraries)
  * [Including Subtrees](#including-subtrees)
- [Limitations of Behavior Trees](#limitations-of-behavior-trees)

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
| Enemy visible? |     | Run away |
+----------------+     +----------+
````
The root task is decomposed into two child tasks, the left child being a condition that an enemy is visible, and the right child being an action which makes the NPC run away.
Each of these actions and conditions can either succeed or fail, which in turn define whether the high-level behavioral task succeeds or fails.

## Composite Tasks ##
Composite tasks provide a standard way to describe relationships between child tasks, such as how and when they should be executed. Contrary to leaf tasks, which are defined by the user, composite nodes are predefined and provided by the behavior tree formalism. They allow you to build branches of the tree in order to organise their sub-tasks (the children). 
Basically, branches keep track of a collection of child tasks (conditions, actions, or other composites), and their behavior is based on the behavior of their children. Unlike actions and conditions, there are normally only a handful of composite tasks because with only a handful of different grouping behaviors we can build very sophisticated behaviors.
Especially, the composite tasks consist of *selectors*, *sequences*, *parallels* and *decorators* that will be described in the next subsessions.

### Selector ###
A selector is a branch task that runs each of its child behaviors in turn. It will return immediately with a success status code when one of its children runs successfully. As long as its children are failing, it will keep on trying. If it runs out of children completely, it will return a failure status code.

Selectors are used to choose the first of a set of possible actions that is successful.
For instance, a selector might represent a character wanting to reach safety. There may be multiple ways to do that: take cover, leave a dangerous area, and find backup. Such a selector will first try to take cover; if that fails, it will leave the area. If that succeeds, it will stop since there's no point also finding backup, as we've solved the character's goal of reaching safety. If we exhaust all options without success, then the selector itself has failed.

A selector node is graphically represented by a question mark.
![selector](https://cloud.githubusercontent.com/assets/2366334/4603480/5f2d3274-516d-11e4-80c5-8e5df55f9fd1.png)

### Sequence ###
A sequence is a branch task that runs each of its child behaviors in turn. It will return immediately with a failure status code when one of its children fails. As long as its children are succeeding, it will keep going. If it runs out of children, it will return in success. 

Sequences represent a series of tasks that need to be undertaken. Each of our reaching-safety actions in the selector example above may consist of a sequence. To find cover we'll need to choose a cover point, move to it, and, when we're in range, play a roll animation to arrive behind it. If any of the steps in the sequence fails, then the whole sequence has failed: if we can't reach our desired cover point, then we haven't reached safety. Only if all the tasks in the sequence are successful we can consider the sequence as a whole to be successful.

A sequence node is graphically represented by an arrow.
![sequence](https://cloud.githubusercontent.com/assets/2366334/4603496/61776ee0-516e-11e4-810a-cdc1a333d50d.png)

### Decorator ###
The name "decorator" is taken from object-oriented software engineering. The decorator pattern refers to a class that wraps another class, modifying its behavior. If the decorator has the same interface as the class it wraps, then the rest of the software doesn't need to know if it is dealing with the original class or the decorator.

In the context of a behavior tree, a decorator is a task that has one single child task and modifies its behavior in some way. You could think of it like a composite task with a single child.

![decorator](https://cloud.githubusercontent.com/assets/2366334/4604011/2bc98188-5183-11e4-952c-fc780e597794.png)

There are many different types of useful decorators:
- **AlwaysFail** will always fail no matter the wrapped task fails or succeeds.
- **AlwaysSucceed** will always succeed no matter the wrapped task succeeds or fails.
- **Include** grafts an external subtree. This decorator enhances behavior trees with modularity and reusability.
- **Invert** will succeed if the wrapped task fails and will fail if the wrapped task succeeds.
- **Limit** controls the maximum number of times a task can be run, which could be used to make sure that a character doesn't keep trying to barge through a door that the player has reinforced.
- **SemaphoreGuard** allows you to specify how many characters should be allowed to concurrently use the wrapped task which represents a limited resource used in different behavior trees (note that this does not necessarily involve multithreading concurrency). This is a simple mechanism for ensuring that a limited shared resource is not over subscribed. You might have a pool of 5 pathfinders, for example, meaning at most 5 characters can be pathfinding at a time. Or you can associate a semaphore to the player character to ensure that at most 3 enemies can simultaneously attack him. This decorator fails when it cannot acquire the semaphore. This allows a select task higher up the tree to find a different action that doesn't involve the contested resource.
- **UntilFail** will repeat the wrapped task until that task fails.
- **UntilSuccess** will repeat the wrapped task until that task succeeds.

There are many more decorators you might want to use when building behavior trees, but I think these are enough for now.


### Parallel ###
A parallel is a special branch task that starts or resumes all children every single time. The parallel task will succeed if all the children succeed, fail if one of the children fail. Note that this is the same policy as the sequence task.

One common use of the parallel task is continually check whether certain conditions are met while carrying out an action. The typical use case: make the game entity react on event while sleeping or wandering.

T.B.C.

# A Simple Example #
T.B.D.

# Behavior Tree API #
Behavior trees are made up of independent tasks, each with its own algorithm and implementation.
At the API level, all of them conform to a basic interface which allows them to call one another without knowing
how they are implemented, see the [Task Class Hierarchy](#task-class-hierarchy) section below.

The API allows you to create behavior tree instances in four different ways:
- from external resources in a simple [text format](#text-format)
- from [libraries](#behavior-tree-libraries)
- by cloning another tree
- programmatically, see [javadoc](http://libgdx.badlogicgames.com/gdx-ai/docs/index.html?com/badlogic/gdx/ai/btree/package-summary.html)

You can freely choose the technique you prefer, but the first two options are recommended as we will see.

## Task Class Hierarchy ##
As you can notice from the figure below, everything is a [Task](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html). Even a [BehaviorTree](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/BehaviorTree.html) is a task. This allows us to easily support the inclusion of sub-trees, as we will see shortly.
[BranchTask](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/BranchTask.html) and [Decorator](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Decorator.html) are the base classes for the composite tasks described in the core concepts above.
[LeafTask](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/LeafTask.html) is the class that you'll have to extend more often in order to implement your actions and conditions.

![task class hierarchy](https://cloud.githubusercontent.com/assets/2366334/4607905/d2890894-5265-11e4-901c-ef775c706df5.png)

## Using Data for Inter-Task Communication ##
To be effective the behavior tree API must allow tasks to share data with one another. The most sensible approach is to decouple the data that behaviors need from the tasks themselves.The API does this by using an external data store for all the data that the behavior tree needs. In AI literature such a store object is known as *blackboard*. Using this external blackboard, we can write tasks that are still independent of one another but can communicate when needed.

In a squad-based game, for example, we might have a collaborative AI that can autonomously engage the enemy. We could write one task to select an enemy (based on proximity or a tactical analysis, for example) and another task or sub-tree to engage that enemy. The task that selects the enemy writes down the selection it has made onto the blackboard. The task or tasks that engage the enemy query the blackboard for a current enemy.

In most games we'll want some characters to have the same behavior, i.e. different instances (or clones as we'll see) of the same behavior tree. In this case each behavior tree instance will require its own blackboard.

At the API level, all the classes in the task hierarchy above are characterized by a generic type parameter `<T>` which is the type of the blackboard. All the tasks of a behavior tree instance have the same generic type argument, `<Blackboard>` for example, so sharing the same blackboard instance.

The [Dog](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/dog/Dog.java) class is a simple example of a blackboard. The action tasks in the same package extend `LeafTask<Dog>`, see the [CareTask](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/dog/CareTask.java) class for example. 

## Text Format ##
The behavior tree text format recognized by the API is a simple and versatile indentation-based format to load behavior trees from an external resource (usually a file) in a [data-driven programming](http://en.wikipedia.org/wiki/Data-driven_programming) style.

All lines in the format have the following syntax (elements within `[` and `]` are optional):
```` 
[[indent] [name [attr:value [...]]] [comment]]
````
where:
- indent is a sequence of spaces/tabs defining the depth of that task in tree
- name is in the form of a Java fully qualified class augmented with the possibility to use the character `?`
- attr is in the form of Java identifier augmented with the possibility to use the character `?`
- value must be a boolean, a number or a quoted string accepting JSON-like escape sequences
- comment starts with `#` and extends up to the first newline character

As you can notice, everything is optional, meaning that an empty line is legal.

Here is a sample behavior tree expressed in our text format:
````
#
# Dog tree
#

# Alias definitions
import bark:"com.badlogic.gdx.ai.tests.btree.dog.BarkTask"
import care:"com.badlogic.gdx.ai.tests.btree.dog.CareTask"
import mark:"com.badlogic.gdx.ai.tests.btree.dog.MarkTask"
import walk:"com.badlogic.gdx.ai.tests.btree.dog.WalkTask"

# Tree definition (note that root is optional)
root
  selector
    parallel
      care urgentProb:0.8
      alwaysFail
        com.badlogic.gdx.ai.tests.btree.dog.RestTask # fully qualified task
    sequence
      bark times:2
      walk
      com.badlogic.gdx.ai.tests.btree.dog.BarkTask # fully qualified task
      mark
````
And the equivalent tree in a graphical form:

![dog tree](https://cloud.githubusercontent.com/assets/2366334/4617800/190bc6c4-5303-11e4-8c52-07470f9a36d7.png)

We have previously mentioned the possibility to use the `?` character in the text format. The question mark is just a syntactic sugar; it gives the user the opportunity to improve readability by making it clear when a leaf task is a condition or an action. For instance, take a look at the tree below.
````
import doorLocked?:"packageName.DoorLockedCondition"
import unlockDoor:"packageName.UnlockDoorAction"
import enterRoom:"packageName.EnterRoomAction"

root
  selector
    sequence
      doorLocked?
      unlockDoor
      enterRoom
    enterRoom
```` 
Notice that you can use the `?` character only inside the alias of an imported task (usually a condition). A fully qualified task name can not contain the `?` because its use is illegal in Java class names. Also, notice that the `?` character is not required for conditions, you might want to use a different naming convention such as `isDoorLocked` or the more verbose `doorLockedCondition`. It's totally up to you. Personally, I prefer the final `?` but it's just a matter of taste.

You can look into the [ParseAndRunTest](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/tests/ParseAndRunTest.java) class for a simple example of how to load a behavior tree into memory from a file.

## Task Attributes and Constraints ##
The framework provides two annotations that are used at runtime by the [BehaviorTreeParser](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/BehaviorTreeParser.html) as metadata to identify task attributes and check task constraints:

- [@TaskConstraint](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskConstraint.html): ideally, any task class should be annotated with this annotation to be properly recognized by the parser. However this is not strictly required. Note that `TaskConstraint` is annotated with `@Inherited` and in turn [Task](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html) is annotated with `@TaskConstraint`. This means that you don't have to annotate a task class with `@TaskConstraint` if the constraint of its superclass is the same. On the other hand, you can use `@TaskConstraint` to "override" the constraint of its superclass. Also, the annotation `TaskConstraint` has two properties:
  * [minChildren](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskConstraint.html#minChildren--) specifies the minimum number of allowed children
  * [maxChildren](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskConstraint.html#maxChildren--) specifies the maximum number of allowed children
- [@TaskAttribute](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskAttribute.html): any task attribute must be annotated with this annotation to be properly recognized by the parser. You can even specify whether the attribute is [required](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskAttribute.html#required--) or optional.



## Behavior Tree Libraries ##
A [BehaviorTreeLibrary](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/BehaviorTreeLibrary.html) is a collection of behavior trees loaded into memory from an external source (usually a file in your application). You can also use it to store named sub-trees that you intend to use in multiple contexts.

It's important to understand that a library contains behavior tree instances known as *archetypes*. You should never use an archetype to run any behaviors on. Any time you need an instance of that behavior tree you ask the library for a copy of the archetype and use the copy. That way you're getting all of the configuration of the tree, but you're getting your own copy. In order to achieve this capability, each task has a [cloneTask](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html#cloneTask--) method that makes a new copy of itself by invoking the protected abstract method [copyTo(Task)](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html#copyTo-com.badlogic.gdx.ai.btree.Task-). The library can then ask the behavior tree or its root task for a clone of itself and have it recursively build us a copy. This would normally be done during the loading of the level, making sure that only the trees that might be needed in that level are loaded and instantiated into archetypes.

This technique presents a simple and effective API. The main advantage of this approach is that if you have a lot of agents having the same behavior (or many common sub-behaviors) the source file doesn't have to be parsed again and again (parsing a tree is something slow producing garbage into memory). On the other hand, the drawback is that you'll have to properly implement the `copyTo()` method in your leaf tasks in order to preserve the task configuration on cloning. The good news is that very often the leaf tasks have no configuration at all (neither children nor parameters), meaning that you don't have to deal with the actual implementation of the method `copyTo()`.

However, you're not forced to use any behavior tree library, unless you want to exploit the include sub-tree capability.
Simple applications whose behavior trees exist as a single instance and not needing a high modularity level can just parse and run their trees directly.

Actually, you're not even forced to load behavior trees from an external source. You can create them programmatically, but mostly for maintenance reasons this is not the recommended approach.

## Including Subtrees ##
As previously said, the [Include](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/decorator/Include.html) task is a decorator that allows you to graft a subtree into another behavior tree instance. This is very useful for reusability when you identify behavioral patterns in your AI agents.

Internally, the include decorator makes use of the singleton [BehaviorTreeLibraryManager](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/BehaviorTreeLibraryManager.html) to get a fresh new instance of the required subtree.

Basically, there are two types of inclusion:
- **Eager inclusion:** It happens when the [lazy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/decorator/Include.html#lazy) attribute of the include decorator is set to `false`. The archetype behavior tree contains these reference nodes, but as soon as we instantiate our full tree it replaces itself with a copy of the sub-tree, built by the library. Notice that the sub-tree is instantiated when the behavior tree is created, ready for a character's use.
- **Lazy inclusion:** It happens when the [lazy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/decorator/Include.html#lazy) attribute of the include decorator is set to `true`. In memory-constrained platforms, or for games with hundreds of AI characters, it may be worth holding off on creating the sub-tree until it is needed, saving memory in cases where parts of a large behavior tree are rarely used. This may be particularly the case where the behavior tree has a lot of branches for special cases: how to use a particular rare weapon, for example. These highly specific sub-trees don't need to be created for every character, wasting memory; instead, they can be created on
demand if the rare situation arises. Actually, the lazy include task creates its child sub-tree at execution time when it is first needed.

Notice that both eager and lazy inclusion are an anomalous decorator that has no child at the archetype level (usually decorators have exactly one child). Especially, the eager include will never execute since it will be replaced by the grafted sub-tree at clone-time, while the lazy include can execute but its child is created and added the first time it executes.

Please, look into the [IncludeSubtreeTest](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/tests/IncludeSubtreeTest.java) class for a simple example of lazy and eager inclusion.

# Limitations of Behavior Trees #
Behavior trees on their own have been a big win for game AI, but you should not consider them as a solution to almost every problem you can imagine in game AI.
As we have seen, behavior trees work great if your character transitions between types of behavior based on the success or failure of certain actions. However, behavior trees make it more difficult to think and design in terms of states. Actually, they tend to be reasonably clunky when representing state-based behaviors such as:
- a character who needs to respond to external events, for example, interrupting a patrol route to suddenly go into hiding or to raise an alarm
- a character that needs to switch strategies when its ammo is looking low

We're not claiming those behaviors can't be implemented in behavior trees, just that it would be cumbersome to do so.

Of course, you can build a hybrid system where characters have multiple behavior trees and use a state machine to determine which behavior tree they are currently running. Using the approach of having behavior tree libraries that we saw above, this provides the best of both worlds. Unfortunately, it also adds considerable extra burden to the AI authors and toolchain developers, since they now need to support two kinds of authoring: state machines and behavior
trees.
