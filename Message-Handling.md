Well-designed games tend to be event driven.  With event handling, entities can make their usual business until an event message is broadcast to them. Then, if that message is pertinent, they can act upon it.
Intelligent agents can use this technique to communicate to each other.

## The Concept of Messages ##

At a core level, a message is a number or an enumerated type. However, the key is that these numbers have a shared meaning among independent systems, thus providing a common interface in which events can be communicated.
At a higher level, a message is contained inside a data structure carrying some additional information such as a _sender_, a _recipient_, a _time stamp_, and possibly some extra data depending on the message itself. This data structure is called **telegram**.
Basically, there are two types of telegrams: 
- **Immediate Telegrams**: They are immediately sent to the recipient. There's not much to say about these messages since their use is rather intuitive.
- **Delayed Telegrams**: They store the time at which they should be delivered. That way the routing system that receives these message objects can retain them until it's time to be delivered, effectively creating timers. These message timers are best used when a game object sends a message to itself, at a later time, to create an internal **event timer**.

The idea is that you can _send a telegram_ to any agent or system in your game.
For example, if a character strikes an enemy, the message `Attacked` could be sent to that enemy and the enemy could respond appropriately to that message event. By having game objects communicate through messages, deep and responsive AI can be achieved fairly easily. Moreover, the technique of messages makes it easy to keep most behaviors event-driven, which is important for efficiency reasons.


## Dispatching a Message ##

The creation, dispatch, and management of telegrams is handled by the singleton class MessageDispatcher.

Whenever an agent needs to send a message, it calls MessageDispatcher.dispatchMessage() like that
````java
	MessageDispatcher.getInstance().dispatchMessage(
			delay,
			sender,
			recipient,
			messageType,
			extraInfo);
````
where _delay_ is expressed in seconds. The MessageDispatcher uses this information to create a Telegram, which it either dispatches immediately (if the given delay is <= 0) or stores in a queue (when the given delay is > 0) ready to be dispatched at the correct time.

### Multiple Recipients ###
If you send a message without specifying the recipient the message will be dispatched to all the agents listening to that message type. Agents can register and unregister their interest in specific message types.
The following methods allow you to manage agent's interest.
````java
// Lets the agent listent to msgCode
MessageDispatcher.getInstance().addListener(agent, msgCode);

// Lets the agent listent to the given selection of msgCodes
MessageDispatcher.getInstance().addListener(agent, msgCode1, msgCode2, ...);

// Removes msgCode from the interests of the agent
MessageDispatcher.getInstance().removeListener(agent, msgCode);

// Removes the given msgCodes from the interests of the agent
MessageDispatcher.getInstance().removeListener(agent, msgCode1, msgCode2, ...);

// Removes all the agents listening to msgCode
MessageDispatcher.getInstance().clearListeners(msgCode);

// Removes all the agents listening to the given selection of msgCodes
MessageDispatcher.getInstance().clearListeners(msgCode1, msgCode2, ...);

// Removes all the agents listening to any message type
MessageDispatcher.getInstance().clearListeners();
````

### Time Granularity ###
To prevent many similar telegrams bunching up in the queue and being delivered en masse, thus flooding an agent with identical messages, you can control **time granularity**.
Delayed telegrams having the same sender, recipient and message type are considered identical when they belong to the same time slot. If time granularity is greater than 0 identical telegrams are not doubled into the queue.
The following code sets the time granularity to 1 second.
````java
	MessageDispatcher.getInstance().setTimeGranularity(1.0f);
````
Of course, the time granularity will vary according to your game. Games with lots of actions producing a high frequency of messages will probably require a smaller slot.
The default granularity is 0.25, i.e. a quarter of a second.
To eliminate time granularity just set it to 0.

### Updating the Dispatcher ###
The queued telegrams are examined each update step by the method MessageDispatcher.dispatchDelayedMessages() which checks the front of the message queue to see if any telegrams have expired time stamps. If so, they are dispatched to their recipient and removed from the queue.
The following call
````java
	MessageDispatcher.getInstance().dispatchDelayedMessages();
````
**must be placed in the game's main update loop** to facilitate the correct and timely dispatch of any delayed messages.

## Receiving a Message ##

When a telegram is received by an agent, its method handleMessage(telegram) is invoked.
This method returns a boolean value indicating whether the message has been handled successfully.

**IMPORTANT NOTE:**
- **Pooling:**
Keep in mind that telegrams are pooled so to limit garbage collection. Also any telegram is automatically released to the pool as soon as it has been dispatched and the handleMessage method of the recipient agent has returned. This means that **you should never keep a reference to the telegram**.

## Telegram Providers ##

In event driven games, new agents might want to gather informations of specific types as soon as they are created (or as soon as they start listening to that type of event) as well as during their whole lifecycle. When registering, a `Telegraph` cannot access some informations without hard references to the sources of those informations :
  - informations carried by `Telegram` dispatched before its registration
  - informations held by other agents 

`TelegramProviders` allow the `MessageDispatcher` to provide newly registered `Telegraph` with immediate `Telegram`. 
Providers can register and unregister their ability to provide informations for specific message types. The following methods allow you to manage provider's abilities :
````java
// Lets the provider respond when a new Telegraph listen to msgCode
MessageDispatcher.getInstance().addProvider (provider, msgCode);

// Lets the provider respond when a new Telegraph listen to msgCode1 or msgCode1 ...
MessageDispatcher.getInstance().addProviders (provider, msgCode1, msgCode2, ...);

// Removes all the providers
MessageDispatcher.getInstance().clearProviders ();

// Removes all the providers responding to new Telegraph listening to msgCode1 or msgCode1 ...
MessageDispatcher.getInstance().clearProviders (msgCode1, msgCode2, ...);

// Removes all the providers responding to new Telegraph listening to msgCode
MessageDispatcher.getInstance().clearProviders (int msgCode);
````

When a new `Telegraph` start listening to a specific type of message, the `TelegramProvider` can decide to provide or not extra information that will be **immediately** delivered to the `Telegraph` by the `MessageDispatcher`.
````java
	Object provideMessageInfo (int msg, Telegraph receiver);
````
... when returning `null`, there's no `Telegram` dispatch to the `Telegraph` by the `TelegramProvider`.