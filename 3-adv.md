# Advanced CloudStack Development

* [CloudStack Bootstrapping](#cloudstack-bootstrapping)
* [Jobs Framework](#jobs-framework)
* [RPC Framework](#rpc-framework)
  * [Agent Framework](#agent-framework)
* [SystemVMs](#systemvms)
* [Self Study](#self-study)

## CloudStack Bootstrapping

Building blocks: spring, managed-context, api, component-api

Manager, ManagerBase
Adapter, AdapterBase
ComponentLifecycle, ComponentLifecycleBase,
ComponentContext


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

(Related: see how usage server starts in `UsageServer.java`)

The `framework/spring/module` implements a spring based module framework
that discovers and constructs the hierarchy of CloudStack module based on the
module.properties file (defined in most CloudStack maven projects). It is
initialized via the `CloudStackContextLoaderListener`.

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Putting+CloudStack+together

## Jobs Framework

AsyncJobManager

## RPC Framework

Cluster and Agent framework
CloudStack clustering, agent LB etc. Agent/Cluster manager

## SystemVMs

### SystemVM Template and systemvm.iso

Packer, build etc.
https://cwiki.apache.org/confluence/display/CLOUDSTACK/SystemVm.iso

### CPVM and SSVM

### Virtual Routers

How it builds, programs etc.
VR python codebase

## Self Study

- API framework:
  - API handling and execution: ApiServlet, ApiServer/ApiServerService, DispatchWorker, DispatchTask, ApiResponseSerializer
  - API jobs execution: ApiDispatcher, ApiAsyncJobDispatcher
  - API security: APIAuthenticationManager, APIAuthenticator, UserAuthenticator, APIChecker
  - Misc: [CallContext](https://cwiki.apache.org/confluence/display/CLOUDSTACK/Using+CallContext), LogContext, ManagedContext, ManagedThreadLocal
- DB framework: GenericDao, GenericDaoBase, TransactionCallback, TransactionLegacy, StateMachine, GenericQueryBuilder, GenericSearchBuilder, Filter, EntityManagerImpl
- CA framework: CAProvider, RootCAProvider, Link, [TLS v1.2](https://tls.ulfheim.net), [TLS v1.3](https://tls13.ulfheim.net/)
- Orchestration engine:
  - Agent related: AgentAttache, ConnectedAgentAttache, DirectAgentAttache, ClusteredAgentAttache, ClusteredDirectAgentAttache, AgentManagerImpl, ClusteredAgentManagerImpl
  - Cluster related: ClusterBasedAgentLoadBalancerPlanner, ClusteredAgentRebalanceService
  - VM related: ClusteredVirtualMachineManagerImpl, VirtualMachineManagerImpl, VirtualMachinePowerStateSyncImpl, VmWork (cloud-engine-components-api)
- Misc Plugin Development:
  - Host and Storage pool allocator plugin: http://docs.cloudstack.apache.org/en/latest/developersguide/alloc.html
  - Storage plugin: http://docs.cloudstack.apache.org/en/latest/developersguide/plugins.html#storage-plugins
- Misc videos:
  - State of Cloud https://www.youtube.com/watch?v=RJMSUDTl6Ds
  - Scalability in CloudStack https://www.youtube.com/watch?v=JqktvtVKnX8 (first half)
- Misc reading:
  - https://cloudierthanthou.wordpress.com/2017/02/22/design-patterns-in-orchestrators-part-1/
  - https://cloudierthanthou.wordpress.com/2017/02/23/design-patterns-in-orchestrators-part-2/
  - https://cloudierthanthou.wordpress.com/2017/06/22/design-patterns-in-orchestrators-transfer-of-desired-state-part-3n/

Misc links:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/201+-+Orchestration+and+Plugins
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Alex%27s+Architecture+Notes
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/VM+Deployment+Planning+and+Resource+Allocation
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/High+Availability+Developer%27s+Guide
