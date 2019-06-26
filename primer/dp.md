# Design Patterns

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

Example Design Patterns implementations in Java:
- https://github.com/rhtyd/hacklab/tree/master/designpatterns/java

## Orchestration Patterns

Recommended reading list (from one of the original CloudStack architects):
- Idempotent operations: https://cloudierthanthou.wordpress.com/2017/02/22/design-patterns-in-orchestrators-part-1/
- Southbound APIs: https://cloudierthanthou.wordpress.com/2017/02/23/design-patterns-in-orchestrators-part-2/
- Transfer of desired state: https://cloudierthanthou.wordpress.com/2017/06/22/design-patterns-in-orchestrators-transfer-of-desired-state-part-3n/
- Network and firewall management:
  - https://cloudierthanthou.wordpress.com/2015/04/09/how-to-manage-a-million-firewalls-part-1/
  - https://cloudierthanthou.wordpress.com/2015/04/10/how-to-manage-a-million-firewalls-part-2/
