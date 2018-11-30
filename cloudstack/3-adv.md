# Advanced CloudStack Development

- Packages and modules
- API discovery, handling etc.
- Design and Architecture
  - Modules and Components
  - Kernel and building blocks
  - Layers and Subsystems
  - IPC, RPC and Communication
  - Dependency Injection
  - Plugin architecture
- Orchestration Layer
- Hypervisor Support
- Networking models
- System VMs and Virtual Routers
- Specialized Plugin: Auth, Hypervisor, Storage, Network

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

## Job Framework?

    AsyncJobManager

Misc:
https://cwiki.apache.org/confluence/display/CLOUDSTACK/VM+Deployment+Planning+and+Resource+Allocation

## Orchestration

https://cwiki.apache.org/confluence/display/CLOUDSTACK/201+-+Orchestration+and+Plugins
https://cwiki.apache.org/confluence/display/CLOUDSTACK/Alex%27s+Architecture+Notes

## HA

https://cwiki.apache.org/confluence/display/CLOUDSTACK/High+Availability+Developer%27s+Guide


## Advanced Development Topics

| Topic | Effort |
| ----- | ------ |
| SystemVM | 5-10 hours |
| Virtual Router | 10-40 hours |
| [Programming Patterns in CloudStack](hack/patterns.md) | 10-40 hours |
| Hypervisor Plugin | 5-10 hours |
| Storage Plugin | 5-10 hours |
| Network Plugin | 5-10 hours |
