# Functional Specification

## Design and Requirements

The following problem can have several similarities with CloudStack's design,
organization units and orchestration. The task is to design the features for
a cafe owner who wants to use CloudStack for his cafe management and
orchestration. For mockups and workflows you may use draw.io or dia.

High level requirements:
  - The coffee shop has an owner (root admin), a manager (domain admin),
    baristas (agent/workers) and its customers (users).
  - Users can place an order using a kiosk either using CloudStack UI or API.
  - Every coffee order is asynchronous for which a unique ID should be returned
    that can be queried.
  - The coffee shop owner (i.e the admin) will create some default coffee
    offerings for their menu for example, flat white, mocha, cappuccino and
    espresso, all of which tune the following [attributes](https://www.quora.com/What-are-the-main-differences-between-a-latte-a-cappuccino-a-mocha-and-a-macchiato) to be treated as boolean and define in ml:
    - Milk foam
    - Steamed milk
    - Espresso
    - Water
    - Chocolate (syrup)
  - The coffee itself can come in three size offerings: small, medium and large
  - Once coffee is available for pickup, the kiosks should broadcast/display
    that it is ready for pickup and if it is not picked by a certain
    configurable time interval (setting) it should be discarded and user would
    lose their purchase.
  - Typical cafe lifecycle:
    - Add a coffee offering for the menu
    - List coffee offering
    - List coffee cup size
    - Add a barista
    - Remove a barista
    - List available coffee orders and pickup an order
  - Coffee has a manager, manager lifecycle and responsibilities:
    - Orchestrate a coffee purchase request
    - Maintain order queue and states
    - Maintain staff, hire/fire based on owner's request
  - A typical user lifecycle:
    - User can place a coffee order with a selected offering from the menu for a
      coffee shop
    - User can query if their order is ready for pickup
    - User can pick their coffee from the available tray

The assumption with the above system is that each barista can handle only one
order at a time (think single core).

Over a period of time and discussions with the customer, the scope and
requirements of the feature request may change. Some additional scope, areas to
consider, anticipate and think about:
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

## CloudStack Functional Spec

Your task now is to read and understand the FS for the above requirements and
then proceed with the implementation.

References:
- CloudStack Design docs (FS): https://cwiki.apache.org/confluence/display/CLOUDSTACK/Design
- FS case study (Dynamic Roles): https://cwiki.apache.org/confluence/display/CLOUDSTACK/Dynamic+Role+Based+API+Access+Checker+for+CloudStack

### APIs

- createCafe(owner only)
- destroyCafe (owner only)
- closeCafe (owner and manager)
- listCafes (everyone)
- createCoffeeOffering (owner only)
- updateCoffeeOffering (owner only)
- deleteCoffeeOfferings (owner only)
- listCoffeeOfferings (everyone)
- listCoffeeSizes (everyone)
- addBarista (owner and manager)
- removeBarista (owner and manager)
- listBaristas (owner and manager)
- orderCoffee (everyone)
- listOrders (owner, manager, and user only by order ID)
- pickOrder (everyone)

### UI

### Service Layer

### DB

### Global Setting and Background Tasks

Global settings:

- `coffee.order.broadcast.interval`: The time after which available orders
  are broadcasted on message bus.
- `coffee.order.max.wait.interval`: The time in seconds after which unpicked
  coffee orders are discarded.

Background tasks:
- To scan and send messages to message bus for available orders.
- To discard unpicked/expired coffee orders.

### IPC

### RPC

### Cafe Framework and Plugins

### Testing
