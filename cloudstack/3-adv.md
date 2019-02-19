# Advanced CloudStack Development

## Maven Build System Overview

- Basic commands
- Structure
- Profiles
- Flags
- Assembler: systemvm.iso

## Core CloudStack Building Blocks

### Framework: spring, managed-context

### Engine: api, component-api

## CloudStack Put Together

Deployment descriptor: WEB-INF/web.xml

The `web.xml` has:
- Servlets (CloudStartupServlet, ApiServlet, ConsoleProxyServlet)
- Configuration (classpath:log4j-cloud.xml)
- Listeners (CloudStackContextLoaderListener)
- Mappings or filters (what url for what servlet)
- Error pages
- Security constraints

The CloudStartupServlet's init() runs
`ComponentContext.initComponentsLifeCycle()` that kickstart the management
server module/component loading, registration, configuration and start.

The `framework/spring/module` implements a spring based module framework
that discovers and constructs the hierarchy of CloudStack module based on the
module.properties file (defined in most CloudStack maven projects). It is
initialized via the `CloudStackContextLoaderListener`.

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Putting+CloudStack+together

## API Layer

Study and refer to following classes that implement the API layer and handle
an API request:

    ApiServlet, ApiServerService, ApiServer, APIAuthenticationManager

Async API scheduling:

    ApiDispatcher, ApiAsyncJobDispatcher, APIChecker, ApiResponseSerializer

CallContext: https://cwiki.apache.org/confluence/display/CLOUDSTACK/Using+CallContext

## Service Layer

### Resource Tags and Metadata

CloudStack supports adding tags and metadata (details) on resources. This can be
done by the create/list/delete tags APIs and the add/list/remove details APIs.
Resource tag is defined by the `ResourceTag` interface, and tags saved in the
`cloud`.`resource_tags` table.

Other related definitions to look at: `Resource`, `Resource.ResourceType`,
`ResourceMetaDataService`, `ResourceDetail`.

## Orchestration Layer

Core, Utils, APIs, Server

https://cwiki.apache.org/confluence/display/CLOUDSTACK/201+-+Orchestration+and+Plugins
https://cwiki.apache.org/confluence/display/CLOUDSTACK/Alex%27s+Architecture+Notes

Misc:
https://cwiki.apache.org/confluence/display/CLOUDSTACK/VM+Deployment+Planning+and+Resource+Allocation
https://cwiki.apache.org/confluence/display/CLOUDSTACK/High+Availability+Developer%27s+Guide

Engine vs framework

### Framework: db, Engine: engine-schema

### Framework: jobs

AsyncJobManager

### Framework: ca

References:
- TLS Connection: https://tls.ulfheim.net/
- TLS 1.3: https://tls13.ulfheim.net/

### Engine: service

### Engine: orchestration

### Framework: cluster

## CloudStack Agent

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Agents+Framework

## SystemVMs

### SystemVM Template

Packer, build etc.

### CPVM

### SSVM

https://cwiki.apache.org/confluence/display/CLOUDSTACK/SystemVm.iso

### Virtual Routers

How it builds, programs etc.
VR python codebase

## Specialized Plugins

### Auth Plugin

### Allocator Plugin

Planners, acls

Etc...

# Distributed Systems Primer

  - Scalability, Availability
  - Latency, fault tolerance, performance
  - Partition and replication
  - Client-Server, agent based
  - CAP, Time and Order
  - Rebalancing, claim ownership
  - Reconciliation, eventual consistency

CloudStack clustering, agent LB etc. Agent/Cluster manager

References and readings:
- System Design Primer: https://github.com/donnemartin/system-design-primer/blob/master/README.md
- Distributed Systems for Fun and Profit: http://book.mixu.net/distsys/single-page.html
- Consistency models: https://jepsen.io/consistency
- Jepsen Analyses: https://jepsen.io/analyses
- CAP and network partition: https://drive.google.com/file/d/15nxAaVXZwNFnJNVvgtKonNbzxNgTUCxP/view
- Further readings:
  - http://henryr.github.io/distributed-systems-readings/
  - http://christophermeiklejohn.com/distributed/systems/2013/07/12/readings-in-distributed-systems.html
  - High Scalability Blog: http://highscalability.com
  - AOSA: http://aosabook.org/en/index.html
  - Fun: https://github.com/danistefanovic/build-your-own-x/blob/master/README.md#build-your-own-operating-system

## Appendix: Programming Patterns in CloudStack

### Design Patterns

Practice and make yourself familiar with GoFs design patterns. Some popular
patterns in CloudStack:

| Design Pattern | Usage in CloudStack |
| --- | --- |
| AbstractFactory and Factory | Several abstract classes and building blocks |
| Adapter | Resource and managers |
| Bridge | Frameworks and their provider plugins |
| Builder | Several building blocks |
| Command | RPC to talk to agents |
| Facade | Various subsystems |
| Observer and State | FSM and events |
| Decorator, Visitor and Flyweight | Network [programming](https://cwiki.apache.org/confluence/display/CLOUDSTACK/Refactoring+Redundant+Virtual+Router+Implementation) |
| Singleton | Managers and Providers instantiated as Spring beans |
| Strategy | Plugins, algorithms, Service Layers |

Example code:
- https://github.com/rhtyd/hacklab/tree/master/designpatterns/java

**Challenge**: Survey the listed design patterns and if you've not solved them
in the past solve them for any example problem of your choice in Java.

### Orchestration Patterns

Recommended reading list (from one of the original CloudStack architects):
- Idempotent operations: https://cloudierthanthou.wordpress.com/2017/02/22/design-patterns-in-orchestrators-part-1/
- Southbound APIs: https://cloudierthanthou.wordpress.com/2017/02/23/design-patterns-in-orchestrators-part-2/
- Transfer of desired state: https://cloudierthanthou.wordpress.com/2017/06/22/design-patterns-in-orchestrators-transfer-of-desired-state-part-3n/
- Network and firewall management:
  - https://cloudierthanthou.wordpress.com/2015/04/09/how-to-manage-a-million-firewalls-part-1/
  - https://cloudierthanthou.wordpress.com/2015/04/10/how-to-manage-a-million-firewalls-part-2/
