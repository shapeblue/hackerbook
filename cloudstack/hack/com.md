# Message Passing and Communication

CloudStack IPC (inter-process communication) is implemented by the
`cloud-framework-ipc` project. However, not all the building blocks and utilities
in `cloud-framework-ipc` are complete or used in the codebase.

Related reference: https://cwiki.apache.org/confluence/display/CLOUDSTACK/FS+-+VMSync+improvement

CloudStack agent-communication is implemented by means of the `command`
design pattern where typically the management server sends Commands to an
indirect/connected agent (such as the CPVM/SSVM/KVM agent), or to direct agents
(such as the xenserver and vmware server resources) and receives `Answer` in
return.

The `cloud-engine-orchestartion` implements `AgentManagerImpl` that manages
agents by means of `AgentAttache`.

CloudStack management server supports two kinds of agents:
- Direct agent: Uses `DirectAgentAttache`, commands are handled by the same JVM
  which runs the management server.
- Indirect/Connect agent: Uses `ConnectedAgentAttache`, agents connect to the
  management server on its service port `8250` and commands are sent to remote
  agent via a custom RPC and custom serialization/deserialization mechanism. For
  connection and communication it uses `NioServer`, `NioClient`,
  `NioConnection`, `Link` as building blocks secured by the `cloud-ca-framework`
  and sends commands wrapped in `Request` by serializing commands to json,
  gzipping it and for answers the process is reversed. The serializing and
  deserializing logic is implemented in `Request` class.

The CloudStack `cloud-agent` implements `Agent` and `AgentShell` classes that
implement a shell layer between a `ServerResource` and the managment server. The
`AgentShell` handles the agent/shell process and connection, while the `Agent`
class facilitates RPC and passing of commands/answers to/from the
`ServerResource`.

## IPC

### Async Callbacks

The `cloud-framework-rpc` provides building blocks to write (nested)
asynchronous code by using callbacks and futures:

```
AsyncCallbackDispatcher
AsyncCallFuture
AsyncCompletionCallback
AsyncRpcContext<T>
```

These building blocks are useful for writing asynchronous code that need callbacks
to execute a method (logic) from a different layer. Its usage is mostly seen in
storage (volume, snapshot, template) related code in CloudStack.

Here is an example on how these building blocks can be used to create an
asynchronous coffee brewing mechanism that may be nested (passed around layers)
and still work with help from callbacks and context.

First define the context class that extends the `AsyncRpcContext<T>` and used to
hold objects:

```java
    private class CreateCoffeeContext<T> extends AsyncRpcContext<T> {
        private final CoffeeMachine coffeeMachine;

        public CreateCoffeeContext(AsyncCompletionCallback<T> callback, CoffeeMachine coffeeMachine) {
            super(callback);
            this.coffeeMachine = coffeeMachine;
        }

        public CoffeeMachine getCoffeeMachine() {
            return this.coffeeMachine;
        }
    }
```

Next, define define an asynchronous callback method that must be called when
the dispatcher (caller) finishes its job and define the async method where the
dispatcher can be called to complete when the task is complete. As an example,
we can the async-callback method to brew coffee using the configured
`CoffeeMachine` plugin after the coffee state transitions to brewing:

```java
    // Note: callback method should be `protected Void` for `Enhancer` to work
    protected Void BrewCoffeeAsyncCallback(AsyncCallbackDispatcher<CoffeeManagerImpl, Coffee> callback, CreateCoffeeContext<Coffee> context) {
        Coffee coffee = callback.getResult();
        context.getCoffeeMachine().brew(coffee);
        stateTransitTo(coffee, Coffee.Event.OrderReady);
        return null;
    }

    public Coffee brewCoffeeAsync(Coffee coffee, AsyncCompletionCallback<Coffee> callback) {
        stateTransitTo(coffee, Coffee.Event.OrderReceived);
        callback.complete(coffee);
        return coffee;
    }

```

The `createCoffee` can next be made to create the async dispatcher (caller)
instance and configure the async callback method to it, before calling the
async-create method:

```java
    @Override
    @ActionEvent(eventType = EventTypes.EVENT_COFFEE_CREATE, eventDescription = "creating coffee", async = true)
    public Coffee createCoffee(CreateCoffeeCmd cmd) {
        // Validations, checks, example code to save Coffee in DB:
        final CoffeeVO coffee = coffeeDao.persist(new CoffeeVO(cmd.getName(), CallContext.current().getCallingAccountId()));
        // Create coffee context object and save any objects that may be useful
        // for the (nested) layers
        CreateCoffeeContext<Coffee> context = new CreateCoffeeContext<>(null, getCoffeeMachine());
        // Create an async call dispatcher that can call a callback/handler once an async job completes
        AsyncCallbackDispatcher<CoffeeManagerImpl, Coffee> caller = AsyncCallbackDispatcher.create(this);
        // The getTarget() enhances the class instance (this)
        // The callback handler method when evoked is intercepted and saved
        caller.setCallback(caller.getTarget().BrewCoffeeAsyncCallback(null, null));
        caller.setContext(context);
        // Call the async method
        return brewCoffeeAsync(coffee, caller);
    }
```

Tip: use these building blocks when you need to have actions performed by an
upper layer where the caller may be far away from the callee.

### Message Bus

:: subscribe, unsubscribe, publish?

MessageBus
MessageHandler
MessageSubscriber
MessageDispatcher
MessageDetector
PublishScope

Inject `MessageBus` in your class to use it:

```java
    @Inject
    private MessageBus messageBus;
```

You can publish, subscribe, unsubscribe on the message bus for a topic (usually
a constant string):

```
    // Publish example
    messageBus.publish(sender, MESSAGE_RESOURCE_CRUD_EVENT, PublishScope.LOCAL, resouceVO);

    // Subscribe example
    messageBus.subscribe(SomeResourceManager.MESSAGE_RESOURCE_CRUD_EVENT, new MessageSubscriber() {
        @Override
        public void onPublishMessage(String senderAddress, String subject, Object args) {
            try {
                // Handle message
            } catch (final Exception e) {
                LOG.error("Caught exception while handling xyz: ", e);
            }
        }
    });
```

Note: `PublishScope.GLOBAL` is not currently implemented to publish between
multiple-management servers.

### Events and Alerts

CloudStack event framework is implemented by the `cloud-framework-events`
project which allows exporting of events to external queues such as RabbitMQ and
Kafka.

Send alerts?

# Agent Framework based RPC

Command-Answer
Agent.send etc.

IAgentShell etc.

NioServer, NioClient, NioConnection, Link
