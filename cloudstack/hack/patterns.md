# Programming Patterns in CloudStack

WIP, under review

## Design Patterns

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

Challenge:
- Write each of the design patterns for any example problem in Java
  (recommended) and/or Python

## Orchestration Patterns

Must read articles from one of the original architects of CloudStack:
- Idempotent operations: https://cloudierthanthou.wordpress.com/2017/02/22/design-patterns-in-orchestrators-part-1/
- Southbound APIs: https://cloudierthanthou.wordpress.com/2017/02/23/design-patterns-in-orchestrators-part-2/
- Transfer of desired state: https://cloudierthanthou.wordpress.com/2017/06/22/design-patterns-in-orchestrators-transfer-of-desired-state-part-3n/

Network and firewall management:
- https://cloudierthanthou.wordpress.com/2015/04/09/how-to-manage-a-million-firewalls-part-1/
- https://cloudierthanthou.wordpress.com/2015/04/10/how-to-manage-a-million-firewalls-part-2/

## Challenge

The following problem can have several similarities with CloudStack's design,
organization units and orchestration. The task is to implement a management
plane and orchestration enginge for a fictitious coffee shop in any language
such as Python, Go etc.

The system is defined by following scope, requirements and specs:
  - The coffee shop has an owner (root admin), a manager (domain admin),
    baristas (agent/workers) and its customers (users).
  - Users can place an order using a kiosk, which the manager processes and
    schedules the order to baristas. Every coffee order is asynchronous for
    which an order ID is returned that can be queried.
  - The coffee shop owner (i.e the admin) will create some default coffee
    offerings for their menu for example, flat white, mocha, cappuccino and
    espresso, all of which tune the following [attributes](https://www.quora.com/What-are-the-main-differences-between-a-latte-a-cappuccino-a-mocha-and-a-macchiato) to be treated as boolean and define in ml:
    - Milk foam
    - Steamed milk
    - Espresso
    - Water
    - Whipped Cream
    - Chocolate Syrup
  - The coffee itself can come in various offerings such as small, medium and
    large
  - Typical cafe lifecycle:
    - Add a coffee offering
    - List coffee offering
    - Add coffee cup size
    - Remove coffee cup size
    - List coffee cup size
    - Add a barista
    - Remove a barista
    - List available coffee orders for pickup
  - Coffee has a manager, manager lifecycle and responsibilities:
    - Orchestrate a coffee purchase request
    - Maintain order queue and states
    - Maintain staff, hire/fire based on owner's request
  - A typical user lifecycle:
    - User orders a coffee with a selected offering
    - User can query if their order is ready for pickup
    - User can pick their coffee from the available tray
  - Example APIs in the system available through RPC or http API and with ACLs:
    - createOffering (owner only)
    - listOffering (everyone)
    - addCoffeeSize (owner only)
    - removeCoffeeSize (owner only)
    - listCoffeeSizes (everyone)
    - addBarista (owner and manager)
    - removeBarista (owner and manager)
    - listBaristas (everyone)
    - orderCoffee (everyone)
    - queryOrder (everyone)
    - pickOrder (everyone)
    - listAvailableOrders (everyone)

The assumption with the above system is that each barista can handle only one
order at a time (think single core).

Some additional tasks, change and scope and requirements to think about:
- How will you refactor and extend your implementation to accomodate a new
  barista type which can handle one or more order (think single vs multicore)?
- How will the system work if each barista to have an artificial latency (not
  all order processed in same amount of time)?
- Can you system accomodate the ability to allow customers to specify a custom
  coffee offering (i.e. make your own coffee, have sugar, spices etc)?
- Can you orchestration system accomodate multiple shops (think multiple
  regions/zones) if the owner were to expand their business?
- Can you system accomodate a rewards program for customers based on their past
  orders (think usage records)?
- Can there be some limit on how many coffees a customer can order (think limits
  and thresholds)
- Can you system have a log for auditing purposes (think events and logging)
- Can your system show capacities and orders for the owner (think dashboard,
  capacities and usage)
