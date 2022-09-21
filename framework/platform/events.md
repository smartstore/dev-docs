---
description: pub/sub system for loosely coupled communication
---

# Events

* The application publishes event messages on different occasions
* e.g.: CustomerSignedIn, CustomerRegistered, OrderPlaced, OrderPaid, Searching, Indexing etc.
* You can react by consuming such events from anywhere you want, e.g. from your custom module
* Event message can be any complex type, no restrictions, no interface, no base class
* ``[`IEventPublisher`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Events/IEventPublisher.cs) publishes events
* ``[`IConsumer`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Events/IConsumer.cs) interface marks a class as consumer/handler/subscriber for one or multiple events
* INFO: No message queuing or persistance: if the app stops working, message gets lost.

## Consuming Events

* The event handler method must be public, non-static, void/Task and be called
  * `Handle` or `HandleEvent` or `Consume` --> Sync event
  * `HandleAsync` or `HandleEventAsync` or `ConsumeAsync` --> Async event
* The first method param is ALWAYS the event message, or [`ConsumeContext<TMessage>`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Events/ConsumeContext.cs)``
* The [`IConsumerInvoker`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Events/IConsumerInvoker.cs) decides how to call the method based on its signature
  * `void` methods are invoked synchronously
  * `Task` methods are invoked asynchronously and awaited
  * ``[`FireForgetAttribute`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Events/FireForgetAttribute.cs) makes the method run in the background without awaiting:
    * This can be advantageous in long running processes, because the current request thread will not _block_
    * WARN: Use this with caution and only if you know what you are doing :-) A class containing a fire & forget consumer should NOT take dependencies on request scoped services, because task continuation happens on another thread and context gets lost. Pass required dependencies as method parameters instead, the consumer invoker will spawn a new private context for the unit of work and resolve dependencies from this context.
* You can declare additional dependency parameters in the handler method, e.g.:
  * Task HandleEventAsync(SomeEvent message, IDbContext db, ICacheManager cache, CancellationToken cancelToken)
* Order does not matter. The invoker will auto-resolve appropriate instances and pass them to the method
* But any unregistered dependency or a primitive type will raise an exception
* Except of `CancellationToken,` which always resolves to app shutdown token
* All types implementing `IConsumer` are auto-discovered on app startup. No need to register in service container.
* The class itself is registered as scoped dependency, so it can also take dependencies in constructor
* TIPP: if multiple handler methods in class --> pass shared deps in class ctor, use method params otherwise
* _SAMPLE_
* _TIPP:_ for module developers --> it is good practice to add a file _Events.cs_ to the root of the module and implement all handler methods within it. The class should be internal. If it gets too large, split/group the methods: either many or just partial classes.

## Publishing events

* _Howto_
* _SAMPLE_
* INFO: don't call the sync `Publish` method, unless you absolutely cannot avoid it. It blocks the thread if any subscriber has real async code.

## Message Bus

* ``[`IMessageBus`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Events/IMessageBus.cs)``
* For inter-server communication between nodes in a web farm
* Gets active when e.g. the REDIS module is installed (which delivers a provider for this)
* Falls back to `NullMessageBus` by default (which does nothing)
* Message can only be of `string` type, no complex types allowed
* It is guaranteed that the server, which published the message, will not consume it.
* _SAMPLE_ (publish and subscribe)
