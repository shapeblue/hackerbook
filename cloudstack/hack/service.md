# CloudStack Service Layer Development

## Declaring Loadable Module

Most CloudStack APIs are handled by a service layer class that usually called
a `XXManagerImpl` or `YYServiceImpl`. For a new feature developed as a plugin,
the plugin needs to export how it should be loaded when management server
starts for which CloudStack has its own Spring based module loading system.

The following is a typical CloudStack module hierarchy where various CloudStack
features implemented as separate maven projects or plugins and usually built as
separate jars (such as hypervisors, networks, storages) are loaded:

```bash
        bootstrap
        └── system
            └── core
                └── allocator
                └── api
                └── backend
                    └── ca
                    └── compute
                        └── kvm
                        └── vmware
                        └── xenserver
                        └── ...
                    └── network
                    └── storage
                    └── discoverer
```

Following this, you need to create a module definition file that declares
how your `feature` should be loaded. For example, at
`plugins/hackerbook/feature/src/main/resources/META-INF/cloudstack/feature`:

```bash
> cat module.properties
name=feature
parent=backend
```

Next, in the same folder you need to declare a context `xml` file where you
declare a java class that can be instantiated and loaded by Spring as a bean.
The name of the context file is usually named as
`spring-nameofyourmodule-context.xml`. Use the module name same as declared
in the `module.properties` file.

```xml
> cat spring-feature-context.xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                      http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="coffeeManager" class="org.apache.cloudstack.feature.CoffeeManagerImpl" >
    </bean>

</beans>

```

The `Spring` framework is widely used in CloudStack primarily for dependency
injection, context, AOP and web (servlet).

Reference readings:
- https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/
- https://dzone.com/articles/how-dependency-injection-di-works-in-spring-java-a
- https://www.journaldev.com/2410/spring-dependency-injection
- https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring
- https://www.tutorialspoint.com/spring
- https://www.javatpoint.com/spring-tutorial

## Manager Implementation

In CloudStack each implementation has a separate API contract class which is
typically a Java interface (and sometimes an abstract class) and separates
how an entity/object/resource is declared and defined.

For the manager, implement a `CoffeeManager` interface that defines a contract
of methods and fields that the implementation class should implement. For
example:

```java
package org.apache.cloudstack.api;

public interface CoffeeManager {
    // define methods here as you progress your implementation
}
```

The manager implementation class would implement the `CoffeeManager` interface
and few other interfaces. The `ManagerBase` defines a typical manager component
that internally is based on `ComponentLifecycleBase`. The `Configurable`
interface is used if the manager needs to export some global settings. The
`PluggableService` is used if the manager needs to export some API commands.

A manager lifecycle has basically three methods:
- `configure`: this is called before a manager class is started and useful to
  configure and instantiate any internal datastructures and configurations.
- `start`: this is called before a manager class is started. Only after the
  manager starts it can then handle any API requests etc.
- `stop`: this is called before a manager is stopped, for example due to
  management server shutdown, and provides means to perform cleanups.

The following manager boilerplate may be used:

```java
package org.apache.cloudstack.feature;

// ...declare imports...

public class CoffeeManagerImpl extends ManagerBase implements CoffeeManager, Configurable, PluggableService {

    private Logger LOGGER = Logger.getLogger(getClass());

    @Override
    public boolean configure(String name, Map<String, Object> params) throws ConfigurationException {
        super.configure(name, params);
        // Add code on how to handle when this is configured
        return true;
    }

    @Override
    public boolean start() {
        // Add code on how to handle when this is started
        return true;
    }

    @Override
    public boolean stop() {
        // Add code on how to handle when this is stopped
        return true;
    }

    @Override
    public List<Class<?>> getCommands() {
        final List<Class<?>> cmdList = new ArrayList<>();
        // add API cmd classes here
        return cmdList;
    }

    @Override
    public String getConfigComponentName() {
        return CoffeeManager.class.getSimpleName();
    }

    @Override
    public ConfigKey<?>[] getConfigKeys() {
        return new ConfigKey[]{
            // Add ConfigKeys here
        };
    }
}
```

Naming convention: depending on the use-case, whether the service/business class
is managing an object or providing a service its name may either have `Manager`
or `Service`. For example, `CoffeeService` and `CoffeeManager`, and their
implementing class may be named `CoffeeServiceImpl` or `CoffeeManagerImpl`. For
the exercise, since the service layer class will be managing the resource
`Coffee` the advice is to use the names `CoffeeManager/CoffeeManagerImpl`.

## Defining Resource Interface

The main resource/object for the feature is `Coffee`, define an interface
that can be used to export contract (methods, enums, constants etc) for that
resource. For example:

```java

package org.apache.cloudstack.api;
public interface Coffee extends InternalIdentity, Identity  {

    enum Size {
        SMALL, MEDIUM, LARGE
    }

    enum Offering {
        Espresso, Cappuccino, Mocha, Latte
    }

    // Define other enum, constants, events, fsm etc.

    String getName();
    String getDescription();
    // Define other methods
}
```

Ensure that you've also used the resource interface to annotate the response
class such as:

```java
@EntityReference(value = Coffee.class)
public class CoffeeResponse extends BaseResponse {
```

We do this to make it easier for the API layer for the id->uuid translation
that we'll discuss in the DB exercise.

## API Request Handling

The typical handler implementation can be to inject the manager in the API
command and in the API's execute() call that dependency (manager/service)
to handle the request by passing arguments or the `cmd` object. For example:

Define the handler in the manager/service interface:

```java
public interface CoffeeManager {
    List<Coffee> listCoffees(ListCoffeesCmd cmd);
    // Other definitions
}
```

The implementation can implement it, such as:

```java
public class CoffeeManagerImpl extends ManagerBase implements CoffeeManager, Configurable, PluggableService {
     // ... code redacted ...
    @Override
    public List<Coffee> listCoffees(ListCoffeesCmd cmd) {
        // Gather inputs from cmd and validate them
        // Search DB based on input params
        // Returned paginated list
        return Collections.emptyList();
    }
```

In the API command, inject the manager/service and use its methods:
```java
public class ListCoffeesCmd extends BaseListCmd {
    // ... redacted code ...
    @Inject
    private CoffeeManager coffeeManager;

    // ... redacted code ...
    @Override
    public void execute() {
        final List<Coffee> coffees = coffeeManager.listCoffees(this);

        // Validate coffee list here

        // Create response, for example in-line or in separate method
        final List<CoffeeResponse> responseList = new ArrayList<>();
        for (final Coffee coffee : coffees) {
            CoffeeResponse response = new CoffeeResponse();
            response.setId(coffee.getUuid());
            response.setName(coffee.getName());
            // other setters etc.
        }
        final ListResponse<CoffeeResponse> coffeeResponses = new ListResponse<>();
        coffeeResponses.setObjectName("coffee");
        coffeeResponses.setResponses(responseList);
        coffeeResponses.setResponseName(getCommandName());
        setResponseObject(coffeeResponses);
    }
```

Ensure that the API is exported by the service layer class:

```java
public List<Class<?>> getCommands() {
    final List<Class<?>> cmdList = new ArrayList<>();
    cmdList.add(ListCoffeesCmd.class);
    ...
```

## Global Settings

ConfigKey usage here...

## Background Tasks

Background Tasks here...

## Exercises

1. Implement the manager module, interface and impl class.

2. Add APIs from previous exercise, build and run the management server and
   verify that you're able to call those APIs using `cmk`.

3. Define a Coffee resource interface and handlers for the APIs in the service
   class.

Challenge: Attempt to find and fix service layer issue(s) from: https://github.com/apache/cloudstack/issues?q=is%3Aissue+is%3Aopen+label%3Abug
