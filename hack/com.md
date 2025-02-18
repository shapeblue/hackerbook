# Message Passing and Communication

CloudStack IPC (inter-process communication) is implemented by the
`cloud-framework-ipc` project. However, not all the building blocks and utilities
in `cloud-framework-ipc` are complete or used in the codebase.

Related reference: https://cwiki.apache.org/confluence/display/CLOUDSTACK/FS+-+VMSync+improvement

## Async Callbacks

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
    protected Void brewCoffeeAsyncCallback(AsyncCallbackDispatcher<CoffeeManagerImpl, Coffee> callback, CreateCoffeeContext<Coffee> context) {
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

## Message Bus

CloudStack messagebus can be used for message/event driven feature
implementation where a subscriber can react to published events/topics.

To use it you can inject `MessageBus` in your class:

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

## Events and Alerts

CloudStack event framework is implemented by the `cloud-framework-events`
project which allows exporting of events to external queues such as RabbitMQ and
Kafka.

Reference: http://docs.cloudstack.apache.org/en/4.11.2.0/adminguide/events.html

Events are generated by async API as well as using `ActionEventUtils`, for
example:

```java
    ActionEventUtils.onActionEvent(userId, accountId, domainId, EventTypes.EVENT_SOME_ACTION, description);
```

The `AlertManager` can be used to send alerts which will email the admin, as well
as create events. For example:

```java
    @Inject
    private AlertManager alertManager;

    // Example usage:
    alertManager.sendAlert(AlertManager.AlertType.ALERT_TYPE_XYZ, zoneId, podId, subject, message);
```

## Agent Framework based RPC

CloudStack uses the `command` design pattern to send commands to a
`ServerResource` resource (directly via shared memory, or indirectly via
network) and these commands are handled by `executeRequest` and an answer is
returned back.

Tip: CPVM/SSVM/KVM agents work as indirect or connected agents.

The `cloud-engine-orchestration` implements `AgentManagerImpl` that manages
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

Some popular `ServerResource` support resources are: KVM
(LibvirtComputingResource), VMWare (VmwareResource), XenServer
(CitrixResourceBase), SSVM (NfsSecondaryStorageResource), and Simulator
(AgentResourceBase).

To use agent based RPC, first define your `Command` class. For example:

```java
public class CoffeeBrewCommand extends Command {
    private Coffee coffee;

    public CoffeeBrewCommand(final Coffee coffee) {
        this.coffee = coffee;
    }

    @Override
    public boolean executeInSequence() {
        return false;
    }
}
```

Next, you can send a command instance using the `AgentManager`.

```java
    @Inject
    private AgentManager agentManager;

    // Example code
    CoffeeBrewCommand command = new CoffeeBrewCommand(coffee);
    Answer answer = null;
    try {
        answer = agentManager.send(hostId, command);
    } catch (AgentUnavailableException e) {
    } catch (OperationTimedoutException e) {
    }
    // process answer
```

The command can be handled by writing a handler method or wrapper class that
handles the command for a `ServerResource`. For example, in case of KVM:

```java
package com.cloud.hypervisor.kvm.resource.wrapper;

@ResourceWrapper(handles = CoffeeBrewCommand.class)
public final class CoffeeCommandWrapper extends CommandWrapper<CoffeeBrewCommand, Answer, LibvirtComputingResource> {
    @Override
    public Answer execute(final CoffeeBrewCommand command, final LibvirtComputingResource libvirtComputingResource) {
        // handle brew-ops
        return new Answer(command);
    }
}
```

Note: Scp/copy the agent/kvm jars to the KVM host(s).

## Exercises

1. Implement the create coffee method with `async` callbacks.

2. Write messagebus handler to create/remove coffee when account is
   create/removed. Send alerts to the admin when a coffee is discarded by the GC
   background task.

3. Refactor one of the `CoffeeMachine` plugins to brew coffee remotely on a
   `ServerResource` using command-answer pattern. You can use KVM or simulator.
