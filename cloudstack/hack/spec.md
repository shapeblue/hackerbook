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

### Service Changes

The feature would require a new service/business layer class that handles
various API requests along with exports APIs, global settings and schedules
background task(s). This may also implement the framework and support discovery
and usage of plugins.

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
  - coffee_id: the coffee ID
  - key: the string key
  - value: the string value

### UI

The feature should be implemented as a UI plugin which shows up as a new
`coffee` tab in the UI. In the tab, users should see a table of coffee orders
with name, offering, size and state. On clicking a coffee, it should show a
details tab. The UI should provide appropriate buttons to create, update and
remove coffee.

### Misc: Global Setting, Background Task and Usage records

Global settings:

- `coffee.ttl.interval`: The max time in seconds after which coffee becomes
  stale, default 600 seconds.
- `coffee.gc.interval`: The interval in seconds for the GC thread to run,
  default 10s.
- `coffee.plugin`: The name of coffee plugin, default is `default`.

Background task: Run based on GC interval and scan coffees across all accounts
and remove coffee that have exceeded the coffee TTL interval.

Usage record:
- Introduce a new coffee usage record type
- Create usage records for coffee per account, introduce a new helper table
- The usage record should have the coffee quantity per account aggregate by
  offering with start/end datetime

### Framework and Brewing Plugins

The framework should create a `CoffeeMaker` interface that implements a `brew()`
method. The plugins can be implemented

### IPC

On creation of any new CloudStack account, a fresh default coffee must be
created and on removal of an account all coffees of that account must be purged.
On coffee creation and removal, events must be emitted.

### RPC

KVM plugin?

### Testing

Marvin test will be written with following cases:

- CRUD tests:
  - Create coffee
  - List coffee
  - Update coffee
  - Remove coffee
- Create coffee and see if GC thread garbage collects it based on global setting
- Change brewing plugin and validate brewing process
