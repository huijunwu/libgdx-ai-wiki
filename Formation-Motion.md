- [Introduction](#introduction)
- [Fixed Formations](#fixed-formations)
- [Scalable Formations](#scalable-formations)
- [Two-Level Formation Motion](#two-level-formation-motion)
- [Multi-Level Formation Motion](#multi-level-formation-motion)
- [Slot Assignment Strategies](#slot-assignment-strategies)
- [The API](#the-api)
- [Dynamic Slots and Plays](#dynamic-slots-and-plays)



## Introduction ##

Many games require groups of characters to move in a coordinated manner.

This section looks at ways to move groups of characters in a cohesive way, having already made the decision that they should move together. This is usually called formation motion.

At its simplest formation motion can consist of moving in a fixed geometric pattern such as a V or line abreast, but it is not limited to that. Formations can also make use of the environment. Squads of characters can move between cover points using formation steering, for example.

Formation motion is used in team sports games, squad-based games, real-time strategy games, and an increasing number of first-person shooters, driving games, and action adventures. It is a simple and flexible technique that is much quicker to write and execute and can produce much more stable behavior than collaborative tactical decision making.


## Fixed Formations ###

The simplest kind of formation movement uses fixed geometric formations. A formation is defined by a set of slots: locations where a character can be positioned. 

One slot is marked as the leader's slot. All the other slots in the formation are defined relative to this slot. Effectively, it defines the "zero" for position and orientation in the formation.

The character at the leader's location moves through the world like any non-formation character would. It can be controlled by any steering behavior, it may follow a fixed path, or whatever else. Whatever the mechanism, it does not take into account the fact that it is positioned in the formation.

The formation pattern is positioned and oriented in the game so as that the leader is located in its slot, facing the appropriate direction. As the leader moves, the pattern also moves and turns in the game. In turn, each of the slots in the pattern move and turn in unison. The position and orientation of each character is determined directly from the formation geometry, without requiring a steering system of its own.

The leader must have limits on the speed it can turn (to avoid outlying characters sweeping round at implausible speeds), and any collision or obstacle avoidance behaviors should take into account the size of the whole formation. In effect, fixed formations are somewhat too rigid and these constraints on the leader's movement make it difficult to use them for anything but very simple formation requirements.

The framework does not support fixed formations because they are so rudimentary and their use in practical use cases is much too limited.
 
 
## Scalable Formations ##

In many situations the exact structure of a formation will depend on the number of characters that are joining it. For example, a defensive circle (characters are around the circumference and their backs are to its center) will be wider with 20 defenders than with 5. With 100 defenders, it may be possible to structure the formation in several concentric rings.

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

When we steer the anchor point directly, it is usually allowed to disregard small-scale obstacles and other characters. However formation's members may take considerably longer to move than expected because they are having to navigate these obstacles. This can lead to the formation and its characters getting a long way out of synch.

A simple solution is to moderate the movement of the formation based on the current positions of the characters in its slots: in effect to keep the anchor point on a leash. If the characters in the slots are having trouble reaching their targets, then the formation as a whole should be held back to give them a chance to catch up.

This can be simply achieved by resetting the steering data of the anchor point at each frame. Its position, orientation, velocity, and rotation are all set to the average of those properties for the characters in its slots. If the anchor point's steering system gets to run first, it will move forward a little, moving the slots forward and forcing the characters to move also. After the slot characters are moved, the anchor point is reined back so that it doesn't move too far ahead.

Because the position is reset at every frame, the target slot position will only be a little way ahead of the character when it comes to steer toward it. Using the arrive behavior will mean that each character is fairly nonchalant about moving such a small distance, and the speed for the slot characters will decrease. This, in turn, will mean that the speed of the formation decreases (because it is being calculated as the average of the movement speeds for the slot characters). On the following frame the formation's velocity will be even less. Over a handful of frames it will slow to a halt.

An offset is generally used to move the anchor point a small distance ahead of the center of mass. The simplest solution is to move it a fixed distance forward, as given by the velocity of the formation:
````
        Pa = Pc + K * Vc
````
where `Pa` is the position of the anchor point, `Pc` and `Vc` are the position and the velocity of the center of mass respectively, and K is a coefficient determined by experimentation.

It is also necessary to set a very high maximum acceleration and maximum velocity for the anchor point. The formation will not actually achieve this acceleration or velocity because it is being held back by the actual movement of its characters.

##### Drift #####
Moderating the formation motion requires that the anchor point of the formation always be at the center of mass of its slots i.e., its average position. Otherwise, if the formation is supposed to be stationary, the anchor point will be reset to the average point, which will not be where it was in the last frame. The slots will all be updated based on the new anchor point and will again move
the anchor point, causing the whole formation to drift across the level.

It is relatively easy, however, to recalculate the offsets of each slot based on a calculation of the center of mass of a formation (the average position of occupied slots). Changing from the old to the new anchor point involves changing
each slot coordinate according to:
````
        Ps[i] = Ps[i] - Pc
````
where `Ps[i]` is the position of slot `i`.

For efficiency, this should be done once and the new slot coordinates stored, rather than being repeated every frame. It may not be possible, however, to perform the calculation offline. Different combinations of slots may be occupied at different times. When a character in a slot gets killed, for example, the slot coordinates will need to be recalculated because the center of mass will have changed.

Drift also occurs when the anchor point is not at the average orientation of the occupied slots in the pattern. In this case, rather than drifting across the level, the formation will appear to spin on the spot. We can again use an offset for all the orientations based on the average orientation of the occupied slots.
As before, changing from the old to the new anchor point involves changing each slot orientation according to:
````
        Os[i] = Os[i] - Oc
````
where `Os[i]` is the orientation of slot `i` and `Oc` is the orientation of the center of mass.

This should also be done as infrequently as possible, being cached internally until the set of occupied slots changes.


## Multi-Level Formation Motion ##

The two-level steering system can be extended to more levels, giving the ability to create formations of formations. This is becoming increasingly important in military simulation games with lots of units; real armies are organized in this way.

We can just extend the concept to support any depth of formation. Each formation has its own anchor point whose steering is managed in turn by another formation. The anchor point is trying to stay in a slot position of a higher level formation.


## Slot Assignment Strategies ##

So far we have assumed that any character can occupy each slot. While this is normally the case, some formations are explicitly designed to give each character a different role.

Slots in a formation can have roles so that only certain characters can fill certain slots. When a formation is assigned to a group of characters (this can even be done by the player), the characters need to be assigned to their most appropriate slots. Whether using slot roles or not, this should not be a haphazard process, with lots of characters scrabbling over each other to reach the formation.

I real world applications, assigning characters to the slots of a formation with roles can become a complex problem. In game applications, a simplification can be used that gives good enough performance.

##### Hard and Soft Roles #####

Imagine a formation of characters in a fantasy RPG game. As they explore a dungeon, the party needs to be ready for action. Magicians and missile weapon users should be in the middle of the formation, surrounded by characters who fight hand to hand.

We can support this by creating a formation with roles. We have three roles: magicians, missile weapon users, and melee (hand to hand) weapon users. Each character has one or more roles that it can fulfill. Characters are only allowed to fill a slot if they can fulfill the role associated with that slot. This is known as a **hard role**.

With hard roles problems can arise when a party is assigned to the formation. For example, if the player wants to create a formation made up of magicians only you don't have enough slots (i.e. roles) to place them and some of them will remain unassigned. We could solve this problem by having many different formations for different compositions of the party. In fact, this would be the optimal solution. Unfortunately, it requires lots of different formations to be designed. If the player can switch formation, this could multiply up to several hundred different designs.

A simpler compromise approach uses **soft roles**: roles that can be broken. Rather than a character having a list of roles it can fulfill, it has a set of values representing how difficult it would find it to fulfill every role. The value is known as the slot cost. To make a slot impossible for a character to fill, its slot cost should be infinite (you can even set a threshold to ignore all slots whose cost is too high; this will reduce computation time when several costs are exceeding). To make a slot ideal for a character, its slot cost should be zero. We can have different levels of unsuitable assignment for one character. For example, magicians might have a very high slot cost for occupying a melee role but a slightly lower cost for missile slots.

We would like to assign characters to slots in such a way that the total cost is minimized. If there are no ideal slots left for a character, then it can still be placed in a non-suitable slot. The total cost will be higher, but at least characters won't be left stranded with nowhere to go.

Notice that soft roles act just like hard roles when the formation can be sensibly filled but don't fail when the wrong characters are available.

##### Slot Assignment Algorithm #####

Slot assignment needs to happen relatively rarely in a game. Most of the time a group of characters will simply be following their slots around. Assignment usually occurs when
- a group of previously disorganized characters are assigned to a formation
- the formation changes its pattern
- formation's members are added or removed

For large numbers of character and slots, the assignment can be done in many different ways. We could simply check each possible assignment and use the one with the lowest slot cost.

Unfortunately, the number of assignments to check gets huge  very quickly. The number of possible assignments of `k` members to `n` slots is given by the permutations formula: 

````
      P(k,n) = n! / (n − k)!
````

For a formation of 20 slots and 20 members, this gives nearly 2500 trillion different possible assignments. Clearly, no matter how infrequently we need to do it, we can't check every possible assignment. 

The assignment problem is an example of a non-polynomial time complete (NP-complete) problem; it cannot be properly solved in a reasonable amount of time by any algorithm.

So gdxAI simplifies the problem by using a heuristic. We won't be guaranteed to get the best assignment, but we will usually get a decent assignment very quickly. The heuristic assumes that a member will end up in a slot best suited to it. We can therefore look at each member in turn and assign it to a slot with the lowest slot cost.

We run the risk of leaving a member until last and having nowhere sensible to put it. GdxAI improves the solution by considering highly constrained members first and flexible members last. The characters are given an ease of assignment value which reflects how difficult it is to find slots for them.

The ease of assignment value is given by:

![ease of assignment formula](https://cloud.githubusercontent.com/assets/2366334/6507482/83180068-c352-11e4-92b7-00eb5ac02968.png)

where `Ci` is the cost of occupying slot `i`, `n` is the number of possible slots, and `k` is a slot-cost limit, beyond which a slot is considered to be too expensive to consider occupying.

Characters that can only occupy a few slots will have lots of high slot costs and therefore a low ease rating. 

The list of characters is sorted according to their ease of assignment values, and the most awkward characters are assigned first. This approach works in the vast majority of cases and is the standard approach for formation assignment.

##### Generalized Slot Costs #####

Slot costs do not necessarily have to depend only on the character and the slot roles. They can be generalized to include any difficulty a character might have in taking up a slot.

If a formation is spread out, for example, a character may choose a slot that is close by over a more distant slot. 

Notice that we're using a slot cost, rather than a slot score (i.e. high is bad and low is good, rather than the other way around). This means that distance can be directly used as a slot cost.

## The API ##

The system consists of a [Formation](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/Formation.html) class that processes a [FormationPattern](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FormationPattern.html) and generates targets for the characters occupying its slots. Slots are assigned to the members through a [SlotAssignmentStrategy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/SlotAssignmentStrategy.html). Formation motion can be moderated by a [FormationMotionModerator](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FormationMotionModerator.html) if needed.

##### Patterns #####
The [FormationPattern](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FormationPattern.html) interface generates the slot offsets for a pattern, relative to its anchor point through the method [calculateSlotLocation](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FormationPattern.html#calculateSlotLocation-com.badlogic.gdx.ai.utils.Location-int-). It does this after being asked for its drift offset, given a set of assignments. In calculating the drift offset, the pattern works out which slots are needed. If the formation is scalable and returns different slot locations depending on the number of slots occupied, it can use the slot assignments passed into the [calculateDriftOffset](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FormationPattern.html#calculateDriftOffset-com.badlogic.gdx.ai.utils.Location-com.badlogic.gdx.utils.Array-) method to work out how many slots are used and therefore what locations each slot should occupy.

Each particular pattern (such as a V, wedge, circle) needs its own instance of a class that implements the `FormationPattern` interface.

##### Formation Members #####
A Character joining a formation must implement the [FormationMember](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FormationMember.html) interface that allows him to know the target [Location](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/utils/Location.html) of his slot.

Since formations are scalable you can add and remove members by using the methods [addMember](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/Formation.html#addMember-com.badlogic.gdx.ai.fma.FormationMember-) and [removeMember](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/Formation.html#removeMember-com.badlogic.gdx.ai.fma.FormationMember-) respectively. Slots are reassigned after you add or remove a member.

##### Slot Assignment Strategies #####
As you could expect any slot assignment strategy must implement the [SlotAssignmentStrategy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/SlotAssignmentStrategy.html) interface mentioned above.
 
The framework supports two kind of strategies:
- without roles through the [FreeSlotAssignmentStrategy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/FreeSlotAssignmentStrategy.html)
- with soft roles through the [SoftRoleSlotAssignmentStrategy](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/SoftRoleSlotAssignmentStrategy.html). This strategy uses a [SlotCostProvider](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/fma/SoftRoleSlotAssignmentStrategy.SlotCostProvider.html) to get the cost of a given pair member/slot.

Currently hard role assignment strategy is not provided by the framework, but you should be able to implement it your self quite easily.

## Dynamic Slots and Plays ##
So far we have assumed that the slots in a formation pattern are fixed relative to the anchor point. A formation is a fixed pattern that can move around the game level.

The framework can be extended to support dynamic formations that change shape over time. Slots in a pattern can be dynamic, moving relative to the anchor point of the formation.

This is useful for introducing a degree of movement when the formation itself isn't moving, for implementing set plays in some sports games, and for using as the basis of tactical movement.

A simple play can be implemented as a pattern whose slots dynamically jump to new locations and the characters use their arrive behaviors to move there. However, in more complex plays where the route taken by the characters is not direct, the slots might need to move along a path.

To support dynamic formations, an element of time needs to be introduced. You should modify the `FormationPattern` interface to take a time value representing the time elapsed since the formation began. 

Unfortunately, this can cause problems with drift, since the formation will have its slots changing position over time. You could extend the system to recalculate the drift offset in each frame to make sure it is accurate. However, Many games that use dynamic slots and set plays do not use two-level steering. For example, the movement of slots in a baseball game is fixed with respect to the field, and in a football game, the plays are often fixed with respect to the line of scrimmage. In this case, there is no need for two-level steering (the anchor point of the formation is fixed), and drift is not an issue, since there's no need to moderate formation motion.

Often, sports titles use techniques similar to formation motion to manage the coordinated movement of players on the field. Some care does need to be taken to ensure that the players do not merely follow their formation oblivious to what's actually happening on the field.

There is nothing to say that the moving slot positions have to be completely pre-defined. The slot movement can be determined dynamically by a coordinating AI routine. At the extreme, this gives complete flexibility to move players anywhere in response to the tactical situation in the game.

In practical use some intermediate solution is sensible. For instance, in a set soccer play for a corner kick, you may decide that only three of the players have pre-defined play motions. The movement of the remaining offensive players will be calculated in response to the movement of the defending team, while the key set play players will be relatively fixed, so the player taking the corner knows where to place the ball. The player taking the corner may wait until just
before he kicks to determine which of the three potential scorers he will cross to. This again will be in response to the actions of the defense. You could, for example, look at the opposing players in the shot cone of each potential scorer and pass to the one with the largest free angle to aim for.