# Basic CloudStack Development

## CloudStack Development 101

- Architecture and Layers
- System Design

**Recommended Reference**:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Development+101
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/How+to+build+CloudStack
- http://docs.cloudstack.apache.org/en/latest/developersguide

## Development Environment

## Getting Code

- Git 101
- Codebase layout and structure

## Setting up IntelliJ IDEA

IDE 101

## Building CloudStack

Maven 101?

Put in sudoers:
Cmnd_Alias CLOUDSTACK = /bin/mkdir, /bin/mount, /bin/umount, /bin/cp, /bin/chmod, /usr/bin/keytool, /bin/keytool

Defaults:bhaisaab !requiretty

bhaisaab ALL=(ALL) NOPASSWD:CLOUDSTACK

## Testing CloudStack

Unit tests and marvin/integration tests

## Simulator Based Development

## MonkeyBox Based Development

Follow the MonkeyBox project to setup a development environment:
https://github.com/rhtyd/monkeybox

## Debugging CloudStack

- Debugging via logs
- Remote debugging with IntelliJ
- Debugging using Visual VM and MAT

## Basic Development Topics

| Topic | Effort |
| ----- | ------ |
| [API Development](hack/api.md) | 2-5 hours |
| [UI Development](hack/ui.md) | 2-5 hours |
| [Service Layer Development](hack/service.md) | 2-5 hours |
| [DB Layer Development](hack/db.md) | 2-5 hours |
| [IPC, Events and message bus](hack/ipc.md) | 2-5 hours |
| [RPC and Agent Framework](hack/rpc.md) | 2-5 hours |
| [Misc: Global Settings, Background Tasks](hack/misc.md) | 2-5 hours |
| [Pluggable Framework and Plugin development](hack/framework.md) | 2-5 hours |
| Upgrade Paths | 2-5 hours |

