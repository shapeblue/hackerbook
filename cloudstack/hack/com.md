# Message Passing and Communication

CloudStack IPC (inter-process communication) is implemented by the
`cloud-framework-ipc` project. However, not all the building blocks and utilities
from this project are complete or used in the codebase.

Article with references on async and message bus:
https://cwiki.apache.org/confluence/display/CLOUDSTACK/FS+-+VMSync+improvement

## IPC and MessageBus

AsyncCallFuture
AsyncCallbackDispatcher
SomeResourceContext<T> extends AsyncRpcContext<T>
AsyncCompletionCallback
AsyncRpcContext

createResourceAsync?

RpcProvider
RPC?
OnwireClassRegistry

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
