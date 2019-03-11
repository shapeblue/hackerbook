# Advanced CloudStack Development

## Spring and CloudStack Put Together

Building blocks: spring, managed-context, api, component-api

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

Core, Utils, APIs, Server

https://cwiki.apache.org/confluence/display/CLOUDSTACK/201+-+Orchestration+and+Plugins
https://cwiki.apache.org/confluence/display/CLOUDSTACK/Alex%27s+Architecture+Notes

Misc:
https://cwiki.apache.org/confluence/display/CLOUDSTACK/VM+Deployment+Planning+and+Resource+Allocation
https://cwiki.apache.org/confluence/display/CLOUDSTACK/High+Availability+Developer%27s+Guide

### DB Framework

Topics: engine-schema, genericdaobase

### Jobs Framework

AsyncJobManager

### CA Framework

References:
- TLS Connection: https://tls.ulfheim.net/
- TLS 1.3: https://tls13.ulfheim.net/

### Engine: service

### Engine: orchestration

### RPC Framework

Cluster and Agent framework
CloudStack clustering, agent LB etc. Agent/Cluster manager

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


