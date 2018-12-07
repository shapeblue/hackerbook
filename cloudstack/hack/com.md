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

## IPC and MessageBus

AsyncCallFuture
AsyncCallbackDispatcher
SomeResourceContext<T> extends AsyncRpcContext<T>
AsyncCompletionCallback
AsyncRpcContext

createResourceAsync?

MessageBus :: subscribe, unsubscribe, publish?
MessageHandler
MessageSubscriber
MessageDispatcher
MessageDetector
PublishScope

## CloudStack Events

CloudStack event framework is implemented by the `cloud-framework-events`
project.

EventBus (Kafka, RabbitMQ, In-memory)

Send alerts?

# Agent Framework based RPC

Command-Answer
Agent.send etc.

IAgentShell etc.

NioServer, NioClient, NioConnection, Link
