- [The Concept of Messages](#the-concept-of-messages)
- [Dispatching a Message](#dispatching-a-message)
- [Multiple Recipients](#multiple-recipients)
- [Return Receipt](#return-receipt)
- [Updating the Dispatcher](#updating-the-dispatcher)
- [Receiving a Message](#receiving-a-message)
- [Telegram Providers](#telegram-providers)
- [Saving and Restoring Pending Messages](#saving-and-restoring-pending-messages)


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

The creation, dispatch, and management of telegrams is handled by the class [MessageDispatcher](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/MessageDispatcher.html).

You can instantiate and use how many dispatchers you want at the same time, but if you need just one in your application you can use the singleton class [MessageManager](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/MessageManager.html) instead. This means that all occurrences of `messageDispatcher` below can be replaced with `MessageManager.getInstance()` that returns the singleton instance of the dispatcher.

Whenever an agent needs to send a message, it calls the [dispatchMessage](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/MessageDispatcher.html#dispatchMessage-float-com.badlogic.gdx.ai.msg.Telegraph-com.badlogic.gdx.ai.msg.Telegraph-int-java.lang.Object-int) method like that
````java
	messageDispatcher.dispatchMessage(
		delay,               // Immediate message if <= 0; delayed otherwise
		sender,              // It can be null
		recipient,           // It can be null, see the "Multiple Recipients" section below
		messageType,         // Any user-defined int code
		extraInfo,           // Optional data accompanying the message; it can be null
		needsReturnReceipt); // Whether the sender needs the return receipt or not
````
where _delay_ is expressed in seconds. The MessageDispatcher uses this information to create a [Telegram](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html), which it either dispatches immediately (if the given delay is <= 0) or stores in a queue (when the given delay is > 0) ready to be dispatched at the correct time.

Note that there are a lot of overloaded versions of the `dispatchMessage` method that can be used for convenience. Just one argument is indeed mandatory, the `messageType`. 

### Multiple Recipients ###
If you send a message without specifying the recipient the message will be dispatched to all the agents listening to that message type. Agents can register and unregister their interest in specific message types.
The following methods allow you to manage agent's interest.
````java
// Lets the agent listent to msgCode
messageDispatcher.addListener(agent, msgCode);

// Lets the agent listent to the given selection of msgCodes
messageDispatcher.addListener(agent, msgCode1, msgCode2, ...);

// Removes msgCode from the interests of the agent
messageDispatcher.removeListener(agent, msgCode);

// Removes the given msgCodes from the interests of the agent
messageDispatcher.removeListener(agent, msgCode1, msgCode2, ...);

// Removes all the agents listening to msgCode
messageDispatcher.clearListeners(msgCode);

// Removes all the agents listening to the given selection of msgCodes
messageDispatcher.clearListeners(msgCode1, msgCode2, ...);

// Removes all the agents listening to any message type
messageDispatcher.clearListeners();
````

### Return Receipt ###
The return receipt feature makes the `MessageDispatcher` instantly send the telegram back to the sender as soon as all receivers have processed the message. Just before the telegram is sent back by the MessageDispatcher some of its fields are updated:
- the field [returnReceiptStatus](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#returnReceiptStatus) is set to [RETURN_RECEIPT_SENT](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#RETURN_RECEIPT_SENT)
- the field [sender](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#sender) is set to the MessageDispatcher
- the field [receiver](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#receiver) is set to the original sender

All the other fields, namely [message](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#message) and [extraInfo](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#extraInfo), remain untouched. 

A typical use case of the return receipt is to let the sender release the `extraInfo` (here we're assuming it's pooled, of course).

Also, it's worth noting that
- the return receipt is normally useful for delayed telegrams. Indeed, for instant messages you can just release the `extraInfo` to the pool after the invocation of the `dispatchMessage` method in the sender's code.
- if the sender is also one of the receivers, it will receive the telegram twice in the same frame: the first time just like all the other receivers with the field `returnReceiptStatus` set to [RETURN_RECEIPT_NEEDED](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#RETURN_RECEIPT_NEEDED), the second time after all the other receivers with the field `returnReceiptStatus` set to [RETURN_RECEIPT_SENT](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegram.html#RETURN_RECEIPT_SENT). The latter is the actual return receipt, so it's when you should release the `extraInfo` object to the pool (or do whatever you need to do). 


### Updating the Dispatcher ###
The queued telegrams are examined each update step by the method [MessageDispatcher.update()](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/MessageDispatcher.html#update--) which checks the front of the message queue to see if any telegrams have expired time stamps. If so, they are dispatched to their recipient and removed from the queue.
The following call
````java
	messageDispatcher.update();
````
**must be placed in the game's main update loop** to facilitate the correct and timely dispatch of any delayed messages.
Notice that the message dispatcher internally calls [GdxAI.getTimepiece()](https://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/GdxAI.html#getTimepiece--) to get the current AI time and properly dispatch delayed messages. This means that
- if you forget to update the timepiece the delayed messages won't be dispatched.
- the timepiece should be updated before the message dispatcher.

## Receiving a Message ##

When a telegram is received by an agent (actually a [Telegraph](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegraph.html)), its method [handleMessage(telegram)](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/Telegraph.html#handleMessage-com.badlogic.gdx.ai.msg.Telegram-) is invoked.
This method returns a boolean value indicating whether the message has been handled successfully.

**IMPORTANT NOTE:**
- **Pooling:**
Keep in mind that telegrams are pooled so to limit garbage collection. Also any telegram is automatically released to the pool as soon as it has been dispatched and the handleMessage method of the recipient agent has returned. This means that **you should never keep a reference to the telegram**.

## Telegram Providers ##

In event driven games, new agents might want to gather informations of specific types as soon as they are created (or as soon as they start listening to that type of event) as well as during their whole lifecycle. When registering, a `Telegraph` cannot access some informations without hard references to the sources of those informations:
- informations carried by `Telegram` dispatched before its registration
- informations held by other agents 

A [TelegramProvider](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/TelegramProvider.html) allows the `MessageDispatcher` to provide newly registered `Telegraph` with immediate `Telegram`. 
Providers can register and unregister their ability to provide informations for specific message types. The following methods allow you to manage provider's abilities:
````java
// Lets the provider respond when a new Telegraph starts listening to msgCode
messageDispatcher.addProvider(provider, msgCode);

// Lets the provider respond when a new Telegraph starts listening to msgCode1, msgCode2, ...
messageDispatcher.addProviders(provider, msgCode1, msgCode2, ...);

// Removes all the providers
messageDispatcher.clearProviders();

// Removes all the providers responding to new Telegraph listening to msgCode
messageDispatcher.clearProviders(msgCode);

// Removes all the providers responding to new Telegraph listening to msgCode1, msgCode2, ...
messageDispatcher.clearProviders(msgCode1, msgCode2, ...);
````

When a new `Telegraph` starts listening to a specific type of message, the `TelegramProvider` can decide to provide or not extra information that will be **immediately** delivered to the `Telegraph` by the `MessageDispatcher`.
````java
	Object provideMessageInfo (int msg, Telegraph receiver);
````
... when returning `null`, there's no `Telegram` dispatch to the `Telegraph` by the `TelegramProvider`.


## Saving and Restoring Pending Messages ##
Some games need to save a snapshot of the level at a given time `T` so it can be loaded later. In this situation you have to serialize and deserialize the status of the queue at time `T`, i.e. its pending messages. 

You can save pending messages with the following code
````java
	messageDispatcher.scanQueue(new PendingMessageCallback() {
		@Override
		public void report (float delay, Telegraph sender, Telegraph receiver, int message, Object extraInfo) {
			// Here you can serialize the pending message.
			// Notice that pending messages are reported in any particular order.
		}
	});
````
To restore pending messages on deserialization you can simply add them to the queue through the [dispatchMessage](http://libgdx.badlogicgames.com/gdx-ai/docs/com/badlogic/gdx/ai/msg/MessageDispatcher.html#dispatchMessage-float-com.badlogic.gdx.ai.msg.Telegraph-com.badlogic.gdx.ai.msg.Telegraph-int-java.lang.Object-) method as usual. 