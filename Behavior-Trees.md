- [Introduction](#introduction)
- [Core Concepts](#core-concepts)
  * [Leaf Tasks](#leaf-tasks)
  * [Composite Tasks](#composite-tasks)
    - [Selector](#selector)
    - [Random Selector](#random-selector)
    - [Sequence](#sequence)
    - [Random Sequence](#random-sequence)
    - [Decorator](#decorator)
    - [Parallel](#parallel)
- [A Simple Example](#a-simple-example)
- [Behavior Tree API](#behavior-tree-api)
  * [Task Class Hierarchy](#task-class-hierarchy)
  * [The Task Superclass](#the-task-superclass)
  * [Using Data for Inter-Task Communication](#using-data-for-inter-task-communication)
  * [Task Attributes and Constraints](#task-attributes-and-constraints)
  * [Text Format](#text-format)
  * [Behavior Tree Libraries](#behavior-tree-libraries)
  * [Including Subtrees](#including-subtrees)
- [Combining Behavior Trees and State Machines](#combining-behavior-trees-and-state-machines)

# Introduction #

For the last few years, behavior trees (BT) are the major formalism used in game industry to build complex AI behaviors. This success comes from the simplicity to understand, use and develop BT by non programmers. 

Behavior trees have been proposed as an improvement over Hierarchical State Machines for designing game AI of non-player characters (NPC). Their advantages over traditional AI approaches are being simple to design and implement, scalability when games get larger and more complex, and modularity to aid reusability and portability.

Behavior trees have been originally pioneered by [robotics](http://en.wikipedia.org/wiki/Robotics) and soon adopted for controlling AI behaviors in commercial games such as first-person-shooter [Halo 2](http://www.gamasutra.com/view/feature/130663/gdc_2005_proceeding_handling_.php) and life-simulation game [Spore](http://chrishecker.com/My_liner_notes_for_spore#Behavior_Tree_AI).

**To better understand behavior trees evaluation, it's recommended that you play with our demo [BehaviorTreeTests](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/BehaviorTreeTests.java) that shows you the status of the tasks at each evaluation step. Also, the demo allows you to run the tree indefinitely at regular intervals or step-by-step, load and save a tree snapshot, reset the whole tree to its initial state. We believe that this is a valuable learning tool for beginners.**

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

Some leaf tasks natively provided by the framework are:
- **Failure** immediately fails.
- **Success** immediately succeeds.
- **Wait** keeps running for the specified amount of time then succeeds.

## Composite Tasks ##
Composite tasks provide a standard way to describe relationships between child tasks, such as how and when they should be executed. Contrary to leaf tasks, which are defined by the user, composite nodes are predefined and provided by the behavior tree formalism. They allow you to build branches of the tree in order to organize their sub-tasks (the children). Basically, branches keep track of a collection of child tasks (conditions, actions, or other composites), and their behavior is based on the behavior of their children.

You can think of composite tasks as the backbone of the behavior tree formalist. Unlike actions and conditions, there are normally only a handful of composite tasks because with only a handful of different grouping behaviors we can build very sophisticated behaviors. Especially, the composite tasks consist of *selectors*, *sequences*, *parallels* and *decorators* that will be described in the next subsessions.

### Selector ###
A selector is a branch task that runs each of its child behaviors in turn. It will return immediately with a success status code when one of its children runs successfully. As long as its children are failing, it will keep on trying. If it runs out of children completely, it will return a failure status code.

Selectors are used to choose the first of a set of possible actions that is successful.
For instance, a selector might represent a character wanting to reach safety. There may be multiple ways to do that: take cover, leave a dangerous area, and find backup. Such a selector will first try to take cover; if that fails, it will leave the area. If that succeeds, it will stop since there's no point also finding backup, as we've solved the character's goal of reaching safety. If we exhaust all options without success, then the selector itself has failed.

A selector node is graphically represented by a question mark.
![selector](https://cloud.githubusercontent.com/assets/2366334/4603480/5f2d3274-516d-11e4-80c5-8e5df55f9fd1.png)

### Random Selector ###
There's a simple variation of the selector task that can make your AI more interesting and varied, the non-deterministic selector. While the standard selector runs each of its children in a strict order defined in advance by who has created the tree, the non-deterministic variation runs its children in a random order. As an example, imagine a dog that wants to rest, it could either sit or lie down. You can quickly implement this behavior by using a random selector with two children, `Sit Down` and `Lie Down`.
![random selector](https://cloud.githubusercontent.com/assets/2366334/10428008/881c282e-70ee-11e5-92b4-9d232b066383.png)

### Sequence ###
A sequence is a branch task that runs each of its child behaviors in turn. It will return immediately with a failure status code when one of its children fails. As long as its children are succeeding, it will keep going. If it runs out of children, it will return in success. 

Sequences represent a series of tasks that need to be undertaken. Each of our reaching-safety actions in the selector example above may consist of a sequence. To find cover we'll need to choose a cover point, move to it, and, when we're in range, play a roll animation to arrive behind it. If any of the steps in the sequence fails, then the whole sequence has failed: if we can't reach our desired cover point, then we haven't reached safety. Only if all the tasks in the sequence are successful we can consider the sequence as a whole to be successful.

A sequence node is graphically represented by an arrow.
![sequence](https://cloud.githubusercontent.com/assets/2366334/4603496/61776ee0-516e-11e4-810a-cdc1a333d50d.png)

### Random Sequence ###
Just like for selector, the non-deterministic variation of the sequence task runs its children in a random order. This comes in handy in certain situations. For instance, suppose you want to burn something. To accomplish this task you need matches and gasoline and, of course, the order you get them is irrelevant. This can be easily implemented through a random sequence with two children: `Get Matches` and `Get Gasoline`.
![random sequence](https://cloud.githubusercontent.com/assets/2366334/10428100/53ad2178-70ef-11e5-8ff0-e1cc28186b13.png)

### Decorator ###
The name "decorator" is taken from object-oriented software engineering. The decorator pattern refers to a class that wraps another class, modifying its behavior. If the decorator has the same interface as the class it wraps, then the rest of the software doesn't need to know if it is dealing with the original class or the decorator.

In the context of a behavior tree, a decorator is a task that has one single child task and modifies its behavior in some way. You could think of it like a composite task with a single child.

![decorator](https://cloud.githubusercontent.com/assets/2366334/4604011/2bc98188-5183-11e4-952c-fc780e597794.png)

There are many different types of useful decorators:
- **AlwaysFail** will always fail no matter the wrapped task fails or succeeds.
- **AlwaysSucceed** will always succeed no matter the wrapped task succeeds or fails.
- **Include** grafts an external subtree. This decorator enhances behavior trees with modularity and reusability. For more detailed information see [Including Subtrees](#including-subtrees)
- **Invert** will succeed if the wrapped task fails and will fail if the wrapped task succeeds.
- **Limit** controls the maximum number of times a task can be run, which could be used to make sure that a character doesn't keep trying to barge through a door that the player has reinforced.
- **Repeat** will repeat the wrapped task a certain number of times, possibly infinite. This task always succeeds when reaches the specified number of repetitions.
- **SemaphoreGuard** allows you to specify how many characters should be allowed to concurrently use the wrapped task which represents a limited resource used in different behavior trees (note that this does not necessarily involve multithreading concurrency). This is a simple mechanism for ensuring that a limited shared resource is not over subscribed. You might have a pool of 5 pathfinders, for example, meaning at most 5 characters can be pathfinding at a time. Or you can associate a semaphore to the player character to ensure that at most 3 enemies can simultaneously attack him. This decorator fails when it cannot acquire the semaphore. This allows a selector task higher up the tree to find a different action that doesn't involve the contested resource.
- **UntilFail** will repeat the wrapped task until that task fails, which makes this decorator succeed.
- **UntilSuccess** will repeat the wrapped task until that task succeeds, which makes this decorator succeed.

There are many more decorators you might want to use when building behavior trees, but I think these are enough for now.


### Parallel ###
A parallel composite task handles "concurrent" behaviors.
It's a special branch task that starts or resumes all children every single time. The actual behavior of this task depends on its policy:
- **Sequence policy**: the parallel task fails as soon as one child fails; if all its children succeed, then the parallel task succeeds. This is the most common policy.
- **Selector policy**: the parallel task succeeds as soon as one child succeeds; if all its children fail, then the parallel task fails.

Notice that the above-mentioned word "concurrent" is in quotes for a reason. Some behavior tree implementations provide multi-threading parallel tasks (concurrently executing children at the same time). Our implementation does not use multi-threading at all. Here a parallel task is just a way to conceptually perform several tasks at once. These tasks still run on the same thread one by one following the specified sequence. That sequence should be irrelevant since they will all happen in the same frame, but it is still sometimes important.

The parallel task with sequence policy acts in a similar way to the sequence task. It has a set of child tasks, and it runs them until one of them fails. At that point, the parallel task as a whole fails. If all of the child tasks complete successfully, the parallel task returns with success. In this way, it looks like the sequence task. Similarly, the parallel task with selector policy acts in a similar way to the selector task.

Regardless of the actual policy, the difference is that both sequence and selector tasks can have only one child in the running status while parallel, which is a "concurrent" task, can have many children in the running status. The demo ["Predators: Parallel vs. Sequence"](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/tests/ParallelVsSequenceTest.java) visually shows this difference through steering behaviors.

Also, when one of the child tasks ends in failure (sequence policy) or success (selector policy), parallel will terminate all of the other children that are still running, so cleanly exiting those children with a cancelled status. This prevents serious problems that could leave the game inconsistent maybe not freeing resources (such as acquired semaphores, see decorators above). In order for this to work, all the tasks of the behavior tree formalism need to be able to receive a termination request.

Here are some typical uses of the parallel task:
- **Non conflicting actions**: the parallel task is most obviously used for sets of actions that can occur at the same time. You might, for example, use parallel to have your character roll into cover while shouting an insult and changing primary weapon. These three actions don't conflict (they wouldn't use the same semaphore, for example), and so you could carry them out simultaneously.
- <a name="condition-checking"></a>**Condition checking**: one very common use of the parallel task is continually check whether certain conditions are met while carrying out an action. For instance, you can make your character react on event while sleeping or wandering. Using parallel tasks to make sure that conditions hold is an important use-case in behavior trees. With it we can get much of the power of a state machine, and in particular the state machine's ability to switch tasks when important events occur and new opportunities arise. Rather than events triggering transitions between states, we can use sub-trees as states and have them running in parallel with a set of conditions. In the case of a state machine, when the condition is met, the transition is triggered. With a behavior tree the behavior runs as long as the condition is met.
- **Group behavior**:  we can use parallel to control the behavior of a group of characters, such as a fire team in a military shooter. While each member of the group gets its own behavior tree for its individual actions (shooting, taking cover, reloading, animating, and playing audio, for example), these group actions are contained in parallel tasks within a higher level selector that chooses the group's behavior. If one of the team members can't possibly carry out their role in the strategy, then the parallel will return in failure and the selector will have to choose another option.


# A Simple Example #
T.B.D.

# Behavior Tree API #
Behavior trees are made up of independent tasks, each with its own algorithm and implementation.
At the API level, all of them conform to a basic interface which allows them to call one another without knowing
how they are implemented, see the [Task Class Hierarchy](#task-class-hierarchy) section below.

The API allows you to create behavior tree instances in four different ways:
- from external resources in a simple [text format](#text-format)
- from [libraries](#behavior-tree-libraries)
- by [cloning another tree](#clone-task)
- programmatically, see [javadoc](http://libgdx.badlogicgames.com/gdx-ai/docs/index.html?com/badlogic/gdx/ai/btree/package-summary.html)

You can freely choose the technique you prefer, but the first two options are recommended as we will see.

## Task Class Hierarchy ##
As you can notice from the figure below, everything is a [Task](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html). Even a [BehaviorTree](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/BehaviorTree.html) is a task. This allows us to easily support the inclusion of sub-trees, as we will see shortly.
[BranchTask](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/BranchTask.html) and [Decorator](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Decorator.html) are the base classes for the composite tasks described in the core concepts above.
[LeafTask](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/LeafTask.html) is the class that you'll have to extend more often in order to implement your actions and conditions.

![task class hierarchy](https://cloud.githubusercontent.com/assets/2366334/4607905/d2890894-5265-11e4-901c-ef775c706df5.png)

## The Task Superclass ##
The [Task](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html) class is the abstract base class of all behavior tree tasks.  All tasks have the following properties:
- **status**: it's the task status which is updated when the task runs or is reset. The possible values are:
  * `FRESH` if the task has never run or has been reset
  * `RUNNING` if the task has not completed and needs to run again
  * `FAILED` if the task returned a failure result
  * `SUCCEEDED` if the task returned a success result
  * `CANCELLED` if the task has been terminated by an ancestor
- **control**: it's the parent of the task. It's set to `null` when the task's status is `FRESH`; set to the parent otherwise.
- **tree**: it's the reference to the behavior tree the task belongs to. Like the control property, it's set to `null` when the task's status is `FRESH`; set to the tree otherwise.

The `Task` class also declares some final methods that, among other things, notify listeners attached to the behavior tree, see [BehaviorTree.Listener](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/BehaviorTree.Listener.html): 
- **running()** called in `run()` to inform the parent that this task needs to run again. The task status is set to `RUNNING`.
- **success()** called in `run()` to inform the parent that this task has finished running with a success result. The task status is set to `SUCCEEDED`.
- **fail()** called in `run()` to inform the parent that this task has finished running with a failure result. The task status is set to `FAILED`.
- **cancel()** terminates this task and all its running children. This method MUST be called only if this task is running. The task status is set to `CANCELLED`.
- **addChild()** adds a child by invoking `addChildToTask()` (see below) and notifies listeners if this task status is not `FRESH`.

The `Task` class also declares the following abstract methods:
- **run()** contains the update logic of the task. The actual implementation MUST call `running()`,  `success()` or `fail()` exactly once.
- **childSuccess()** called when one of the children of the task succeeds.
- **childFail()** called when one of the children of the task fails.
- **childRunning()** called when one of the children of the task needs to run again.
- **addChildToTask()** adds a child to the list of this task's children. It is called by the final method `addChild()` mentioned above.
- **getChildCount()** returns the number of children of the task.
- **getChild()** returns the child at the given index.

The are a few other non-final non-abstract methods that subclasses often override:
- **start()** called when the task is entered, just before `run()` is invoked.
- **end()** called when the task is exited through `success()`, `fail()` or `cancel()`. This means that this task's status has just been set to `SUCCEEDED`, `FAILED` or `CANCELLED` respectively.
- **reset()** resets this task to make it restart from scratch on next run. The task is cancelled if it was running. In any case the task status is set to `FRESH`.
- <a name="clone-task"></a>**cloneTask()** clones this task to a new one. If you don't specify a clone strategy through [TASK_CLONER](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html#TASK_CLONER) the new task is instantiated via reflection and `copyTo()` is invoked. In this case, properly overriding the `copyTo()` method in each task is developer's responsibility. This gives you opportunity to target the GWT backend. On the other hand, `cloneTask()` will use the non-null `TASK_CLONER` instance and `copyTo()` won't be invoked. For instance, if you don't care about GWT, you can let [Kryo](https://github.com/EsotericSoftware/kryo) make a deep copy for you.


## Using Data for Inter-Task Communication ##
To be effective the behavior tree API must allow tasks to share data with one another. The most sensible approach is to decouple the data that behaviors need from the tasks themselves.The API does this by using an external data store for all the data that the behavior tree needs. In AI literature such a store object is known as *blackboard*. Using this external blackboard, we can write tasks that are still independent of one another but can communicate when needed.

In a squad-based game, for example, we might have a collaborative AI that can autonomously engage the enemy. We could write one task to select an enemy (based on proximity or a tactical analysis, for example) and another task or sub-tree to engage that enemy. The task that selects the enemy writes down the selection it has made onto the blackboard. The task or tasks that engage the enemy query the blackboard for a current enemy.

In most games we'll want some characters to have the same behavior, i.e. different instances (or clones as we'll see) of the same behavior tree. In this case each behavior tree instance will require its own blackboard.

At the API level, all the classes in the task hierarchy above are characterized by a generic type parameter `<T>` which is the type of the blackboard. All the tasks of a behavior tree instance have the same generic type argument, `<Blackboard>` for example, so sharing the same blackboard instance.

The [Dog](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/dog/Dog.java) class is a simple example of a blackboard. The action tasks in the same package extend `LeafTask<Dog>`, see the [CareTask](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/dog/CareTask.java) class for example. 

## Task Attributes and Constraints ##
The framework provides two annotations that are used at runtime by the [BehaviorTreeParser](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/BehaviorTreeParser.html) as metadata to identify task attributes and check task constraints:

- [@TaskConstraint](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskConstraint.html): ideally, any task class should be annotated with this annotation to be properly recognized by the parser. However this is not strictly required. Note that `TaskConstraint` is annotated with `@Inherited` and in turn [Task](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html) is annotated with `@TaskConstraint`. This means that you don't have to annotate a task class with `@TaskConstraint` if the constraint of its superclass is the same. On the other hand, you can use `@TaskConstraint` to "override" the constraint of its superclass. Also, the annotation `TaskConstraint` has two properties:
  * [minChildren](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskConstraint.html#minChildren--) specifies the minimum number of allowed children; defaults to `0`
  * [maxChildren](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskConstraint.html#maxChildren--) specifies the maximum number of allowed children; defaults to `Integer.MAX_VALUE`
- [@TaskAttribute](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskAttribute.html): any task attribute must be annotated with this annotation to be properly recognized by the parser. This annotation has two properties:
  * [name](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskAttribute.html#name--) specifies the name of the attribute; if omitted the field's name is used instead. This gives you the capability to use attribute names that are illegal field names in Java, like for instance reserved keywords such as `switch`, `case`, `class` and so on  
  * [required](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/annotation/TaskAttribute.html#required--) specifies whether the attribute is mandatory or optional; defaults to `false`

## Text Format ##
The behavior tree text format recognized by the API is a simple and versatile indentation-based format to load behavior trees from an external resource (usually a file) in a [data-driven programming](http://en.wikipedia.org/wiki/Data-driven_programming) style.

All lines in the text format follow the grammar below (elements within `[` and `]` are optional):
```` 
line = [[indent] [guardableTask] [comment]]
guardableTask = [guard [...]] task
guard = '(' task ')'
task = name [attr:value [...]] | subtreeRef
````
where:
- _indent_ is a sequence of spaces/tabs defining the depth of the task specified by _guardableTask_
- _guardableTask_ is a sequence of optional _guards_ followed by a _task_
- _guard_ is just a _task_ within parentheses `(` and `)`
- _task_ is either a task name with attributes or a subtree reference 
- _name_ identifies the task; it's either a Java fully qualified class name or an imported alias in the form of Java identifier augmented with the possibility to use the character `?`, see `import` in the example below
- _attr_ is the attribute name in the form of Java identifier
- _value_ is the attribute's value that must be one of the following based on the attribute's Java type:
  * `true` or `false` literal for types `boolean`and `Boolean`
  * any number literal for primitive types `byte`, `short`, `int`, `long`, `float`, `double` and their respective boxed Java types
  * double quoted string literal (accepting JSON-like escape sequences) for types `char`, `Character`, `String`, `Enum` and `Distribution`
  * `null`for any non-primitive Java type
- _subtreeRef_ is the name of an internal subtree in the form of a Java identifier preceded by the symbol `$` 
- _comment_ starts with `#` and extends up to the first newline character


As you can notice, everything is optional, meaning that an empty line is legal.
 
Also, notice the special use of strings in case of enumerations and distributions:
- enums: the string is the non-case-sensitive name of the enum constant like, for instance, "sequence" and "selector" for the policy attribute of the parallel task.
- distributions: are a comma-separated string of the form "distributionType,arg1,arg2,..." where 
  * distributionType can be `constant`, `uniform`, `gaussian` and `triangular`
  * the number of arguments identify the overloaded constructor to use

  Built-in distributions are represented by distribution classes in package [com.badlogic.gdx.ai.utils.random](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/utils/random/package-frame.html). User defined distributions are supported by customizing the [DitributionAdapters](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/DistributionAdapters.html) instance provided to the parser. Also, it's worth mentioning that, as a shortcut, constant distributions can be directly represented by the corresponding number. For instance, `repeat times:2` is equivalent to `repeat times:"constant,2"`.

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
      bark times:"uniform,1,3"  # the type of attribute times is a IntegerDistribution 
      walk
      com.badlogic.gdx.ai.tests.btree.dog.BarkTask # fully qualified task
      mark
````
And the equivalent tree in a graphical form:

![dog tree](https://cloud.githubusercontent.com/assets/2366334/4617800/190bc6c4-5303-11e4-8c52-07470f9a36d7.png)

A more complex behavior tree with guards and subtree references can be found [here](https://github.com/jsjolund/GdxDemo3D/blob/master/android/assets/btrees/dog.btree). That behavior tree models a dog that freely moves all around the garden until the owner calls him to play with the stick.

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


## Behavior Tree Libraries ##
A [BehaviorTreeLibrary](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/BehaviorTreeLibrary.html) is a collection of behavior trees loaded into memory from an external source (usually a file in your application). You can also use it to store named sub-trees that you intend to use in multiple contexts.

It's important to understand that a library contains behavior tree instances known as *archetypes*. You should never use an archetype to run any behaviors on. Any time you need an instance of that behavior tree you ask the library for a copy of the archetype and use the copy. That way you're getting all of the configuration of the tree, but you're getting your own copy. In order to achieve this capability, the `Task` class provides [cloneTask](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html#cloneTask--) described above. The library can then ask the behavior tree or its root task for a clone of itself and have it build us a copy. This would normally be done during the loading of the level, making sure that only the trees that might be needed in that level are loaded and instantiated into archetypes.

This technique presents a simple and effective API. The main advantage of this approach is that if you have a lot of agents having the same behavior (or many common sub-behaviors) the source file doesn't have to be parsed again and again (parsing a tree is something slow producing garbage into memory). On the other hand, the drawback is that you'll have to properly implement the `copyTo()` method in your leaf tasks in order to preserve the task configuration on cloning. Alternatively, you can set a task cloner instance (see [Task.TASK_CLONER](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/Task.html#TASK_CLONER)) using a third-party library like [Kryo](https://github.com/EsotericSoftware/kryo) behind the scene.

However, you're not forced to use any behavior tree library, unless you want to exploit the include sub-tree capability.
Simple applications whose behavior trees exist as a single instance and not needing a high modularity level can just parse and run their trees directly.

Actually, you're not even forced to load behavior trees from an external source. You can create them programmatically, but mostly for maintenance reasons this is not the recommended approach.

## Including Subtrees ##
As previously said, the [Include](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/decorator/Include.html) task is a decorator that allows you to graft a subtree into another behavior tree instance. This is very useful for reusability when you identify behavioral patterns in your AI agents.

Internally, the include decorator makes use of the singleton [BehaviorTreeLibraryManager](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/utils/BehaviorTreeLibraryManager.html) to get a fresh new instance of the required subtree.

Basically, there are two types of inclusion:
- **Eager inclusion:** It happens when the [lazy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/decorator/Include.html#lazy) attribute of the include decorator is set to `false` (default). The archetype behavior tree contains these reference nodes, but as soon as we instantiate our full tree it replaces itself with a copy of the sub-tree, built by the library. Notice that the sub-tree is instantiated when the behavior tree is created, ready for a character's use.
- **Lazy inclusion:** It happens when the [lazy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/btree/decorator/Include.html#lazy) attribute of the include decorator is set to `true`. In memory-constrained platforms, or for games with hundreds of AI characters, it may be worth holding off on creating the sub-tree until it is needed, saving memory in cases where parts of a large behavior tree are rarely used. This may be particularly the case where the behavior tree has a lot of branches for special cases: how to use a particular rare weapon, for example. These highly specific sub-trees don't need to be created for every character, wasting memory; instead, they can be created on
demand if the rare situation arises. Actually, the lazy include task creates its child sub-tree at execution time when it is first needed.

Notice that both eager and lazy inclusion are an anomalous decorator that has no child at the archetype level (usually decorators have exactly one child). Especially, the eager include will never execute since it will be replaced by the grafted sub-tree at clone-time, while the lazy include can execute but its child is created and added the first time it executes.

Please, look into the [IncludeSubtreeTest](https://github.com/libgdx/gdx-ai/blob/master/tests/src/com/badlogic/gdx/ai/tests/btree/tests/IncludeSubtreeTest.java) class for a simple example of lazy and eager inclusion.

# Combining Behavior Trees and State Machines #
Behavior trees on their own have been a big win for game AI, but you should not consider them as a solution to almost every problem you can imagine in game AI.
As we have seen, behavior trees work great if your character transitions between types of behavior based on the success or failure of certain actions. However, behavior trees make it more difficult to think and design in terms of states. Actually, they tend to be reasonably clunky when representing state-based behaviors such as:
- a character who needs to respond to external events, for example, interrupting a patrol route to suddenly go into hiding or to raise an alarm
- a character that needs to switch strategies when its ammo is looking low

We're not claiming those behaviors can't be implemented in behavior trees, just that it would be cumbersome to do so. We have already seen how to mimic state machines in behavior trees, see [Parallel Task](#parallel) and in particular [Condition Checking](#condition-checking).

Of course, you can build a hybrid system where behavior trees and state machines cooperate with each other, providing the best of both worlds. There are 2 common approaches:

1. Characters have multiple behavior trees and use a state machine to determine which behavior tree they are currently running. An example of this approach can be found inside the open source project [GdxDemo3D](https://github.com/jsjolund/GdxDemo3D), see the class [DogCharacter.java](https://github.com/jsjolund/GdxDemo3D/blob/master/core/src/com/mygdx/game/objects/DogCharacter.java)

2. Certain tasks in the tree run an internal state machine. 

What follows is just one possible implementation of the second approach.
````java
public class FsmTask extends LeafTask<MyEntity> {

	StateMachine<FsmTask> sm;

	public FsmTask () {
		this.sm = new DefaultStateMachine<FsmTask>(this);
	}

	@Override
	public void start () {
		sm.setInitialState(FsmTaskState.STATE_1);
	}

	@Override
	public void run () {
		sm.update();
		switch ((FsmTaskState)sm.getCurrentState()) {
		case SUCCESS:
			success();
			break;
		case FAIL:
			fail();
			break;
		default:
			running();
			break;
		}
	}

	@Override
	protected Task<MyEntity> copyTo (Task<MyEntity> task) {
		return task;
	}

	public enum FsmTaskState implements State<FsmTask> {

		STATE_1() { // A certain state
			@Override
			public void update (FsmTask smTask) {
				smTask.sm.changeState(MathUtils.randomBoolean(.1f) ? SUCCESS : STATE_2);
			}
		},

		STATE_2() { // A certain state
			@Override
			public void update (FsmTask smTask) {
				smTask.sm.changeState(MathUtils.randomBoolean(.1f) ? FAIL : STATE_1);
			}
		},

		SUCCESS(), // Terminal state indicating success

		FAIL(); // Terminal state indicating failure

		@Override
		public void enter (FsmTask smTask) {
		}

		@Override
		public void update (FsmTask smTask) {
		}

		@Override
		public void exit (FsmTask smTask) {
		}

		@Override
		public boolean onMessage (FsmTask smTask, Telegram telegram) {
			return false; // We don't use messaging in this example
		}
	}

}
````
