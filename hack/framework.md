# CloudStack Framework and Plugin Development

A typical CloudStack framework would define a policy and provide building
blocks, utilities and perhaps export global settings etc and define a plugin
interface that would have methods to extend and implement a mechanism.

## Plugin Development

Your service/manager implementation class can be used for any framework (policy)
implementation. In addition, you need to define a plugin interface that
can implement some mechanism (usually a bunch of methods). The interface needs
to extend the `Adapter` interface which is used throughout CloudStack for plugin
implementation.

For example:

```java
package org.apache.cloudstack.api;

public interface CoffeeMachine extends Adapter {
    Coffee brew(Coffee coffee);
    String getName();
}
```

Next, you can implement the plugin(s) in the same feature maven project or as a
separate maven project. The plugin implementation class need to implement the
defined plugin interface, and may extend the `AdapterBase` abstract class.

For example:

```java
public class DummyCoffeeMachine extends AdapterBase implements CoffeeMachine {
    @Override
    public Coffee brew(Coffee coffee) {
        try {
            Thread.sleep(5000L);
        } catch (InterruptedException ignored) {
        }
        return coffee;
    }

    @Override
    public String getName() {
        return "dummy";
    }
}
```

Whether the plugin class is implemented in your feature maven project or as a
separate maven project, define a bean so it may be discovered and consumed by
the framework. For example:

```xml
    <bean id="dummyCoffeeMachine" class="org.apache.cloudstack.feature.DummyCoffeeMachine" />
```

Notes:
- In CloudStack, the plugin class is typically named as a `Provider`.
- The plugin may also export its own global settings by implementing
  `Configurable`. It may export its own APIs etc as well.

## Plugin Discovery and Usage

Once you've your plugins implemented, you need to define a registry of plugins
based on the plugin interface, then such a registry can be used to inject/set
a list of discovered plugin (beans implementing a certain plugin interface)
to your framework service/manager impl class.

Spring can be used to create a registry, usually of type `ExtensionRegistry`,
and use `RegistryLifecycle` to discover and add the plugin (bean) to the
registry based on the type of class (the plugin interface).

For example:

```xml
    <bean id="coffeeMachineRegistry" class="org.apache.cloudstack.spring.lifecycle.registry.ExtensionRegistry">
        <property name="orderConfigDefault" value="inbuilt" />
    </bean>

    <bean class="org.apache.cloudstack.spring.lifecycle.registry.RegistryLifecycle">
        <property name="registry" ref="coffeeMachineRegistry" />
        <property name="typeClass" value="org.apache.cloudstack.api.CoffeeMachine" />
    </bean>
```

Finally, the registry can be used to inject/set the list of discovered plugin
beans to the framework service/manager impl class. For example:

```xml
    <bean id="coffeeManager" class="org.apache.cloudstack.feature.CoffeeManagerImpl">
        <property name="coffeeMachines" value="#{coffeeMachineRegistry.registered}" />
    </bean>
```

```java
public class CoffeeManagerImpl extends ManagerBase implements CoffeeManager, Configurable, PluggableService {
    // .. code redacted ..
    List<CoffeeMachine> coffeeMachines = new ArrayList<>();
    // the following setter will be used by Spring to set the discovered plugins
    public void setCoffeeMachines(final List<CoffeeMachine> coffeeMachines) {
        this.coffeeMachines = coffeeMachines;
    }
```

Note: in case of implementing an external plugin, the
`org.apache.cloudstack.spring.lifecycle.registry.RegistryLifecycle` bean may be
re-defined on the plugin's context xml for the bean to get discovered and
injected in the manager class. Alternatively, you can define a hierarical plugin
dependency mechanism that will be a cleaner approach, see the
`OutOfBandManagementDriver` and the plugins as an example.

The framework could read a `ConfigKey` that defines the plugin selected for a
scope (global, zone, cluster etc) and use the configured plugin to carry out
an action (mechanism), while it can implement the policy. For finding the plugin
by name, the plugin typically should export its name or some identifier and the
framework can create an internal map of string->plugin to find and use the
configured plugin.

For example:
```java
    // Define map of string -> plugin
    Map<String, CoffeeMachine> coffeeMachineMap = new HashMap<>();
    // .. code redacted ..
    // Example code of how a map can be built at the time your framework starts
    public boolean start() {
        coffeeMachineMap.clear();
        for (final CoffeeMachine machine : coffeeMachines) {
            coffeeMachineMap.put(machine.getName(), machine);
        }
        return true;
    }
    // Example code of writing a plugin getter based on `ConfigKey`
    private CoffeeMachine getCoffeeMachine() {
        final String coffeeMachinePlugin = CoffeeMachinePlugin.value();
        if (coffeeMachineMap.containsKey(coffeeMachinePlugin)) {
            return coffeeMachineMap.get(coffeeMachinePlugin);
        }
        throw new CloudRuntimeException("Invalid coffee machine configured!");
    }
    // Example code of how configured plugin can be used to carry out task
    public Coffee createCoffee(CreateCoffeeCmd cmd) {
        // Validations here
        final CoffeeVO coffee = coffeeDao.persist(new CoffeeVO(cmd.getName(), CallContext.current().getCallingAccountId()));
        // Policy: Change state to reflect ongoing operation
        coffee.setState(Coffee.CoffeeState.Brewing);
        coffeeDao.update(coffee.getId(), coffee);
        // Mechanism: Brew coffee using configured Coffee Machine plugin
        getCoffeeMachine().brew(coffee);
        // Policy: Update state to brewed
        coffee.setState(Coffee.CoffeeState.Brewed);
        coffeeDao.update(coffee.getId(), coffee);
        return coffee;
    }
```

Reference examples of features that implement framework and plugins:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Secure+Agent+Communications
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Out-of-band+Management+for+CloudStack
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Host+HA

## Exercises

1. Implement at least two `CoffeeMachine` plugins, with at least one of them
   implemented as a separate maven project in whose module.properties your
   feature should be the parent. Implement the `ConfigKey` for the
   coffee machine plugin name and use it to drive brewing operations.

2. Write unit tests for the classes.
