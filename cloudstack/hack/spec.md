# Specification

## Requirements

Coffee is a beverage that CloudStack needs to support, a typical coffee life
cycle involves it to be created, updated, listed, removed etc. Coffee can be
brewed by hypervisors however a framework should allow developers to write
custom coffee/brewing plugins. The coffee should have a maximum time to live,
a garbage-collection thread to remove stale coffees. The feature should be
available through UI.

## Functional Spec

Your task now is to read and understand the above requirements and then proceed
with the implementation based on a functional specification.

References:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Adding+new+features+and+design+documents
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Design+Document+Template
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Design

FS references and examples:
- CloudStack Design docs (FS): https://cwiki.apache.org/confluence/display/CLOUDSTACK/Design
- FS case study (Dynamic Roles): https://cwiki.apache.org/confluence/display/CLOUDSTACK/Dynamic+Role+Based+API+Access+Checker+for+CloudStack

The feature can be implemented as a CloudStack plugin at
`plugins/hackerbook/feature` which should also implement a framework and two
coffee machine plugins:
- `default`: the in-built cloudstack brewer machine, implemented at
  `plugins/hackerbook/default`
- `kvm`: the brewer machine that brews coffee on a KVM host, implemented at
  `plugins/hackerbook/kvm`

A `Coffee` should follow a finite-state-machine (FSM) with following states:

- Brewing: Coffee is order is sent
- Brewed: Coffee is brewed
- Ready: Coffee is ready for serving
- Removed: Coffee is removed
- Discarded: Coffee is put in trash

The `size` of coffee can be as follows:
- `SMALL`
- `MEDIUM`
- `LARGE`

The following coffee `offering` should be supported:
- Espresso
- Cappuccino
- Mocha
- Latte

The following sections document high-level implementation details.

### APIs

| API | Arguments | Description |
| --- | --------- | ----------- |
| createCoffee | name, offering, size, details | Creates coffee |
| listCoffees | id, offering, size | Lists coffee by id, type and/or size |
| updateCoffee | id, size, details | Updates coffee order that is not ready yet |
| removeCoffee | id, ids | Removes coffee by id or ids |

The details are string->string map of any arbitrary key/value to note any
details of each coffee order.

### DB Schema

Following tables should be added with appropriate DAO and VO classes:

- coffee:
  - id: auto_increment ID for the table
  - uuid: the UUID for a coffee order
  - name: name on the coffee order
  - offering: the offering/type of the coffee
  - size: size of coffee
  - state: the state of Coffee
  - account_id: the user account ID
  - domain_id: the domain ID
  - created: the creation timestamp
  - removed: the removal timestamp

- coffee_details:
  - id: auto_increment ID for the table
  - coffee_id: the coffee ID
  - name: the string key
  - value: the string value
  - display: whether to show/hide the detail in the API response

### Service Changes

The feature would require a new service/business layer class that handles
various API requests along with exports APIs, global settings and schedules
background task(s). This may also implement the framework and support discovery
and usage of plugins.

Global settings:

- `coffee.ttl.interval`: The max time in seconds after which coffee becomes
  stale, default 600 seconds.
- `coffee.gc.interval`: The interval in seconds for the GC thread to run,
  default 10s.
- `coffee.machine.plugin`: The name of coffee machine plugin, default is `inbuilt`.

Background task: Run based on GC interval and scan coffees across all accounts
and remove coffee that have exceeded the coffee TTL interval.

### Pluggable CoffeeMaker

Implement a framework that allows plugin creation based on a `CoffeeMaker`
interface that implements a `brew()` method. Implement two plugins, an inbuilt
default coffeemaker that performs no-operation (maybe just log) and a separate
maven project based lazy coffeemaker that randomly sleeps in `brew()`.

### Message Passing: IPC and RPC

On creation of any new CloudStack account, a fresh default coffee must be
created and on removal of an account all coffees of that account must be purged.
On coffee creation and removal, publish event on the message bus. On coffee
discarded by GC thread, send an alert.

Create a `CoffeeMachine` plugin that brews coffee via a random agent such as the
CPVM/SSVM agent or the KVM agent.

### Coffee Usage Records

Introduce a new coffee usage record type and implement usage parser for the
coffee resource that generates usage records aggregated per account by the
number of coffees ordered.

### UI

The feature should be implemented as a UI plugin which shows up as a new
`coffee` tab in the UI. In the tab, users should see a table of coffee orders
with name, offering, size, state and account. On clicking a coffee, it should
show a details tab that lists the attributes of the coffee. The UI should
provide appropriate buttons to create, update and remove coffee.

### Testing

Marvin test will be written with following cases:

- CRUD tests:
  - Create coffee
  - List coffee
  - Update coffee
  - Remove coffee
- Create coffee and see if GC thread garbage collects it based on global setting
- Change brewing plugin and validate brewing process

### Packaging

Make the coffee feature and any of its plugins packaged separately as a separate
rpm/deb package.
