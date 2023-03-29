# CloudStack API Development

## Project Skeleton

For the purpose of the exercise, you may implement the feature as a separate
maven project under `plugins`:

    mkdir -p plugins/hackerbook/feature

To the `feature` directory, add a maven project `pom.xml` file:

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cloud-plugin-hackerbook-feature</artifactId>
    <name>Apache CloudStack Plugin - HackerBook Coffee Feature</name>
    <parent>
        <groupId>org.apache.cloudstack</groupId>
        <artifactId>cloudstack-plugins</artifactId>
        <version>4.15.1.0-SNAPSHOT</version>
        <relativePath>../../pom.xml</relativePath>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.apache.cloudstack</groupId>
            <artifactId>cloud-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cloudstack</groupId>
            <artifactId>cloud-utils</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

Note: please change the version suitably as per the base-branch you're using.

The pom.xml has `cloud-api` and `cloud-utils` artifacts as dependencies to
use/extend/implementing API interfaces and use any CloudStack utility classes.

Note: CloudStack master branch's `version` may change, please fix accordingly.

Add the project to CloudStack's plugin pom.xml to get your newly added maven
project (feature) built along with other CloudStack artifacts:

```java
--- a/plugins/pom.xml
+++ b/plugins/pom.xml
@@ -73,6 +73,8 @@
         <module>event-bus/rabbitmq</module>

+        <module>hackerbook/feature</module>

```

Also, add the `feature` maven project to CloudStack's `client/pom.xml` which
builds and bundles various CloudStack artifacts (jars) into a single uberjar:

```java
--- a/client/pom.xml
+++ b/client/pom.xml
@@ -218,6 +218,11 @@
             <artifactId>cloud-plugin-network-vsp</artifactId>
             <version>${project.version}</version>
         </dependency>
+        <dependency>
+            <groupId>org.apache.cloudstack</groupId>
+            <artifactId>cloud-plugin-hackerbook-feature</artifactId>
+            <version>${project.version}</version>
+        </dependency>
```

Now, when you build and run CloudStack your `feature` will be part of the
management server.

Next, per the standard maven convention you'll need to add directories/code per
the following hierarchy:

```bash
        feature
        ├── pom.xml # maven project config
        ├── target  # build directory
        └── src
            └── main
                └── java
                    └── org.apache.cloudstack
                        └── api
                            └── command  # for API command classes
                            └── response # for API response classes
                        └── feature      # for feature classes
                            └── dao      # for feature DAO classes
                └── resources
                    └── resources/META-INF/cloudstack/feature
                        └── module.properties
                        └── spring-feature-context.xml
            └── test
                └── java
                    └── org.apache.cloudstack.feature # for feature unit tests
```

The spring module skeleton and setup will be discussed in the next exercise.

`License notice`: all files contributed to Apache CloudStack should have the
Apache License 2.0 header. See existing files for reference and example.

## Introduction

CloudStack has two API service ports on the default API path
`host:<port>/client/api`:
- 8080 (default): the authenticated API service port.
- 8096: the unauthenticated API service port as defined by the
  `integration.api.port` global setting. It is disabled by default on production
  installations.

CloudStack has two types of (REST-like, query-based end-user/admin) APIs:

- Synchronous:
  - Extends `BaseCmd` class or a child class.
  - Blocking execution of API until an API response is returned.
- Asynchronous:
  - Extends `BaseAsyncCmd` or `BaseAsyncCreateCmd` class or a child class.
  - Creates an async job and returns a job ID. The API response can be checked
    by pollable the `queryAsyncJobResult` API providing it the `jobid`.

A CloudStack API is a class that encapsulates an API request parameters, a
common pattern in CloudStack is to pass an API object to business/service layer.

**Suggested API examples**:
https://github.com/apache/cloudstack/tree/master/api/src/main/java/org/apache/cloudstack/api/command/admin/acl

References:
- http://cloudstack.apache.org/api.html
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+API+Development
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Annotations+use+in+the+API
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+API+Coding+Guidelines
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/How+To+Generate+CloudStack+API+Documentation
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/List*+API+commands+rules
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+IAM+guidelines+for+API+and+Service+Layer
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Coding+conventions

## API implementation

Depending on the type of API you want to implement, create an API class
with its name same or similar to the API name. For example, for the API
`createCoffee` you may create a CreateCoffeeCmd.java, and for `listCoffees`
ListCoffeesCmd.java etc.

Every API is based on two classes (sometimes reusable): a request class and a
response class.

Each API class needs to have an `APICommand` annotation on the class that is
used to export metadata about the API such as the `name`, `description` etc.
Each API class needs to also declare an API response class which is a class that
captures a response of the API. The `since` can have information about
CloudStack version in which the API was introduced. You may also declare API
security doc parameters `requestHasSensitiveInfo` and
`responseHasSensitiveInfo`. Finally, the `authorized` parameter controls which
type of user account may be allowed to execute the API.

Example API code:

```java
@APICommand(name = "myAPI",
            description = "My API short description here",
            responseObject = MyAPIResponse.class,
            since = "4.xx.yy",
            requestHasSensitiveInfo = false, responseHasSensitiveInfo = false,
            authorized = {RoleType.Admin, RoleType.ResourceAdmin, RoleType.DomainAdmin, RoleType.User})
public class MyAPICmd extends BaseCmd {
    public static final Logger LOG = Logger.getLogger(MyAPICmd.class);

```

Next, API can have parameters that can be defined using the `Parameter`
annotation that can defined several attributes of an API parameter such as the
parameter `name`, `description`, `required` etc. Please explore the `Parameter`
interface for full list of attributes.

Example parameter code:

```java
    /////////////////////////////////////////////////////
    //////////////// API parameters /////////////////////
    /////////////////////////////////////////////////////

    @Parameter(name = ApiConstants.ID,
               type = CommandType.UUID,
               required = false,
               entityType = MyAPIResponse.class,
               description = "ID of my resource")
    private Long id;
```

`CommandType` defines the type of the API parameter. The API layer uses this
annotation and declared metadata to validate an API request, for example when
the `type` is BOOLEAN it may try to convert the argument input to a `boolean`
value etc. The following API command types are supported per the
`BaseCmd::CommandType` enum:

```java
CommandType {
    BOOLEAN, DATE, FLOAT, DOUBLE, INTEGER, SHORT, LIST, LONG, OBJECT, MAP, STRING, TZDATE, UUID
}
```

Just like an API, an API `Parameter` may also declare its own `authorized`
field. The API parameters can also define `validations` to use one of the
commonly used API validators, for example:

```java
validations = {ApiArgValidator.NotNullOrEmpty}
validations = {ApiArgValidator.PositiveNumber}
```

Next, the API can define accessors (getters usually) for the parameters. For
example:

```java
    /////////////////////////////////////////////////////
    /////////////////// Accessors ///////////////////////
    /////////////////////////////////////////////////////

    public Long getId() {
        return id;
    }
```

The API implementation is a group of methods that exports
the account ID of the resource owner on which the API is
acted up (`getEntityOwnerId`) and the `execute()` method that handles the API
request. The `getEntityOwnerId()` method can also make use of `CallContext`
utility to get information about the current thread/execution context. For
example, get the account ID that made the API request by using
`CallContext.current().getCallingAccountId()`.
Method to export the API name (`getCommandName`) is not needed now in 4.18 branch or later, after changes introduced in https://github.com/apache/cloudstack/pull/7022.

Reference reading: https://cwiki.apache.org/confluence/display/CLOUDSTACK/Using+CallContext

Example API implementation:

```java
    /////////////////////////////////////////////////////
    /////////////// API Implementation///////////////////
    /////////////////////////////////////////////////////

    @Override
    public long getEntityOwnerId() {
        return Account.ACCOUNT_ID_SYSTEM;
    }

    @Override
    public void execute() {
        // logic to handle API request

        final MyAPIResponse response = new MyAPIResponse();
        // logic to setup the API response object
        response.setResponseName(getCommandName());
        response.setObjectName("object-name");
        setResponseObject(response);
    }
```

### Asynchronous APIs

Asynchronous APIs in CloudStack also need to export the event type (also see
`EventTypes` class) and description information. Such APIs generally have three
phases: API request is received, API execution is asynchronously started by the
job framework, and the API execution finishes and the response is saved. They
typically need to implement these methods:

```java

    @Override
    public String getEventType() {
        return EventTypes.EVENT_XYZ;
    }

    @Override
    public String getEventDescription() {
        return "string description usually with action details and entity ids";
    }
```

An asynchronous API extending the `BaseAsyncCreateCmd` or child class usually
also implements a `create()` method which is run before the `execute()` and
allows creation of an resource entity (usually in the database) before the API
executes. Have a look at the `BaseAsyncCmd` and `BaseAsyncCreateCmd` classes
for overridable methods.

```java
    @Override
    public void create() {
        Resources res = someService.createResource(this);
        if (res != null) {
            this.setEntityId(res.getId());
            this.setEntityUuid(res.getUuid());
        }
        ...
```

CloudStack async APIs can also export events that usually describe the state
of execution (Created, Scheduled, Started, Completed). For this, the async API
implementation's `getEventType` and `getEventDescription` are used by the event
framework to publish these events on the event bus. The handler method defined
in the service/manager class can also define an annotation `@ActionEvent` to
export event type and description metadata that gets captured by the
`ActionEventInterceptor` class by means of `spring-aop`.

```java
    @Override
    @ActionEvent(eventType = EventTypes.EVENT_COFFEE_CREATE, eventDescription = "creating coffee", async = true)
    public Coffee createCoffee(CreateCoffeeCmd createCoffeeCmd) {
        // Logic to create coffee
```

Reference reading:
- http://docs.cloudstack.apache.org/en/4.11.2.0/adminguide/events.html
- https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop

## API response implementation

An API response class typically extends `BaseResponse` and contains response
attributes with `@Param` and `@SerializedName` annotations that define the
serialized attribute/key name and their description. This metadata is used
by CloudStack build system to generate `apidocs` (see in tools/apidocs). The
class may sometimes have a `@EntityReference` annotation to mark the type of
resource the response class represents. Use this annotation when you've a VO
class in future exercise, that implements the resource interface.

An API response class may typically be reused by a resource's list/create/update
APIs and generally contains setters (and sometimes getters). For example:

```java
@EntityReference(value = MyResource.class)
public class MyAPIResponse extends BaseResponse {
    @SerializedName(ApiConstants.ID)
    @Param(description = "the ID of my resource")
    private String id;

    public void setId(String id) {
        this.id = id;
    }
```

### API UUID Translation

Most CloudStack resources/objects have a unique uuid (string), and in
the database they also have a `bigint` ID. The UUID command type allows APIs
with both integer and uuid (string) IDs to be translated and validated to a
CloudStack resource and set that resource's ID to the `Long` field. This
translation is done with help of the `@Parameter` `entityType` field
which generally is a `Response` class having an `@EntityReference` annotation
that declares an interface class which typically implements a `VO` (view object)
class declaring a `@Table`. This way, for each parameter the API layer can
try to find the resource from a database table by the passed `uuid` value and
perform the translation and validation. We'll revisit how API layer work in
detail in future chapters.

### API Docs

When adding a new set of APIs around a resource, the build around `apidocs`
may fail. This is because API docs may not know how to categorize those new
APIs. For this, add a the resource name (for example `Coffee`):

```python
--- a/tools/apidoc/gen_toc.py
+++ b/tools/apidoc/gen_toc.py
@@ -190,7 +190,8 @@ known_categories = {
     'Sioc' : 'Sioc',
-    'Diagnostics': 'Diagnostics'
+    'Diagnostics': 'Diagnostics',
+    'Coffee': 'Coffee'
     }
```

When you build CloudStack, API docs are generated at
`tools/apidoc/target/xmldoc/html`.

## Exercises

1. Implement the following APIs based on the spec, under
   `org.apache.cloudstack.api.command` package:
- createCoffee (extend BaseAsyncCreateCmd)
- listCoffee (extend BaseListCmd)
- updateCoffee (extend BaseAsyncCmd)
- removeCoffee (extend BaseAsyncCmd, use SuccessResponse as response class)

2. Implement API response class `CoffeeResponse` that represents a Coffee
   resource under `org.apache.cloudstack.api.response` package.

3. Write basic unit tests for the classes. (IntelliJ: Ctrl+Shift+t to
   create/browse unit test of a java class)

Challenge: Attempt and fix any upstream CloudStack API related issue(s):
https://github.com/apache/cloudstack/labels/component%3Aapi
