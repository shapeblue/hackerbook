# Advanced CloudStack Development

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

## Orchestration Layer

https://cwiki.apache.org/confluence/display/CLOUDSTACK/201+-+Orchestration+and+Plugins
https://cwiki.apache.org/confluence/display/CLOUDSTACK/Alex%27s+Architecture+Notes


Misc:
https://cwiki.apache.org/confluence/display/CLOUDSTACK/VM+Deployment+Planning+and+Resource+Allocation

https://cwiki.apache.org/confluence/display/CLOUDSTACK/High+Availability+Developer%27s+Guide

### Job Framework

AsyncJobManager

Core, Utils, APIs, Server

Frameworks: DB, IPC, CA, Cluster, Events, Security, Spring, Jobs, Managed Context

CloudStack Engine: api, component-api, orchestartion, network, schema, service, storage

## CloudStack Agent

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Agents+Framework

## SystemVM Template

Packer, build etc.

## SystemVMs: CPVM, SSVM

https://cwiki.apache.org/confluence/display/CLOUDSTACK/SystemVm.iso

## Virtual Routers

How it builds, programs etc.
VR python codebase

# Distributed Systems

  - Scalability, Availability
  - Latency, fault tolerance, performance
  - Partition and replication
  - Client-Server, agent based
  - CAP, Time and Order
  - Rebalancing, claim ownership
  - Reconciliation, eventual consistency

CloudStack clustering, agent LB etc. Agent/Cluster manager

## Specialized Plugins

### Auth Plugin

### Allocator Plugin

Etc...
