- [Introduction](#introduction)
- [Fixed Formations](#fixed-formations)
- [Scalable Formations](#scalable-formations)
- [Two-Level Formation Motion](#two-level-formation-motion)
- [Multi-Level Formation Motion](#multi-level-formation-motion)
- [Slot Roles and Slot Assignment](#slot-roles-and-slot-assignment)


## Introduction ##

Many games require groups of characters to move in a coordinated manner.

This section looks at ways to move groups of characters in a cohesive way, having already made the decision that they should move together. This is usually called formation motion.

At its simplest formation motion can consist of moving in a fixed geometric pattern such as a V or line abreast, but it is not limited to that. Formations can also make use of the environment. Squads of characters can move between cover points using formation steering, for example.

Formation motion is used in team sports games, squad-based games, real-time strategy games, and an increasing number of first-person shooters, driving games, and action adventures. It is a simple and flexible technique that is much quicker to write and execute and can produce much more stable behavior than collaborative tactical decision making.


## Fixed Formations ###

The simplest kind of formation movement uses fixed geometric formations. A formation is defined by a set of slots: locations where a character can be positioned. 

One slot is marked as the leader's slot. All the other slots in the formation are defined relative to this slot. Effectively, it defines the "zero" for position and orientation in the formation.

The character at the leader's location moves through the world like any non-formation character would. It can be controlled by any steering behavior, it may follow a fixed path, or whatelse. Whatever the mechanism, it does not take into account the fact that it is positioned in the formation.

The formation pattern is positioned and oriented in the game so as that the leader is located in its slot, facing the appropriate direction. As the leader moves, the pattern also moves and turns in the game. In turn, each of the slots in the pattern move and turn in unison. The position and orientation of each character is determined directly from the formation geometry, without requiring a steering system of its own.

The leader must have limits on the speed it can turn (to avoid outlying characters sweeping round at implausible speeds), and any collision or obstacle avoidance behaviors should take into account the size of the whole formation. In practice, these constraints on the leader's movement make it difficult to use this kind of formation for anything but very simple formation requirements.

The framework does not support fixed formations because they are so rudimentary and their use in practical use cases is much too limited.
 
 
## Scalable Formations ##

In many situations the exact structure of a formation will depend on the number of characters that are joining it. For example, a defensive circle () will be wider with 20 defenders than with 5. With 100 defenders, it may be possible to structure the formation in several concentric rings.

Usually scalable formations do not specify an explicit list of slot positions and orientations, but rather they use a function to dynamically return the slot locations given the total number of characters in the formation. When members are added to or removed from a formation, the formation  accommodates them, changing its distribution of slots accordingly. 

The framework supports exactly this kind of dynamically scalable formations.


## Two-Level Formation Motion ##

This technique uses a geometric formation, defined as a fixed pattern of slots, and each character uses the slot at a target location for an arrive behavior. Characters can have their own collision avoidance behaviors and any other compound steering required.

This is two-level steering because there are two steering systems in sequence: first the leader steers the entire formation pattern, and then each character in the formation steers to stay in the pattern.

As long as the leader does not move at maximum velocity, each character will have the chance to stay in its slot while taking into account the surroundings. The slot that a character is trying to reach may be briefly impossible to achieve, but its steering algorithm ensures that it still behaves sensibly.

##### Replacing the Leader with an Anchor Point #####
At a first glance, having a leader responsible for guiding the formation might look like a good idea, but actually it's not. For example, if the leader needs to move sideways to avoid an obstacle, then all the slots in the formation will also lurch sideways and every other character will lurch sideways to stay with its slot. This can look odd because the leader's actions are mimicked by the other characters, although they are largely free to cope with obstacles in their own way.

Instead the formation is moved around by an invisible leader, the **anchor point**, which is a separate steering system that is controlling the whole formation, but none of the individuals. This is the second level of the two-level formation motion.

Because this new leader is invisible, it does not need to worry about small obstacles, bumping into other characters, or small terrain features. The invisible leader will still have a fixed location in the game, and that location will be used to lay out the formation pattern and determine the slot locations for all the proper characters. The location of the leader's slot in the pattern will not correspond to any character, however. 

Having a separate steering system for the formation typically simplifies implementation: 
- There is no need to worry about making one character take over as leader if another one dies
- The steering for the anchor point is often simple. Outdoors, we might only need to use a single high-level arrive behavior, for example, or maybe a path follower. In indoor environments the steering will still need to take account of large scale obstacles, such as walls. A formation that passes straight through into a wall will strand all its characters, making them unable to follow their slots.                                                          

##### Moderating the Formation Movement #####
So far information has flowed in only one direction: from the formation to its members. With a two-level steering system, this causes problems. The formation could be steering ahead, oblivious to the fact that its members are having problems keeping up. When the formation was being led by a leader, this was less of a problem, because difficulties faced by the other characters in the formation were likely to also be faced by the leader.

When we steer the anchor point directly, it is usually allowed to disregard small-scale obstacles and formation's members. However the characters in the formation may take considerably longer to move than expected because they are having to navigate these obstacles. This can lead to the formation and its characters getting a long way out of synch.

A simple solution is to moderate the movement of the formation based on the average position and average orientation of the characters in its slots: in effect to keep the anchor point on a leash. If the characters in the slots are having trouble reaching their targets, then the formation as a whole is held back to give them a chance to catch up. This is done internally by the framework.

Usually it is necessary to set a very high maximum acceleration and maximum velocity for the anchor point. The formation will not actually achieve this acceleration or velocity because it is being held back by the actual movement of its characters.


## Multi-Level Formation Motion ##

The two-level steering systemcan be extended to more levels, giving the ability to create formations of formations. This is becomingly increasingly important in military simulation games with lots of units; real armies are organized in this way.

We can just extend the concept to support any depth of formation. Each formation has its own anchor point whose steering is managed in turn by another formation. The anchor point is trying to stay in a slot position of a higher level formation.


## Slot Roles and Slot Assignment ##

T.B.D.