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
        <version>4.12.0.0-SNAPSHOT</version>
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
                └── java # contains Java source code and packages
                    └── org.apache.cloudstack.feature
                └── resources # contains resouces
                    └── resources/META-INF/cloudstack/feature
                        └── module.properties
                        └── spring-feature-context.xml
            └── test
                └── java # contains Java unit tests
                    └── org.apache.cloudstack.feature
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

**Suggested API examples**:
https://github.com/apache/cloudstack/tree/master/api/src/main/java/org/apache/cloudstack/api/command/admin/acl

## API implementation

Depending on the type of API you want to implement, create an API class
with its name same or similar to the API name. For example, for the API
`createCoffee` you may create a CreateCoffeeCmd.java, and for `listCoffees`
ListCoffeesCmd.java etc.

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
@APICommand(name = MyAPICmd.APINAME,
            description = "My API short description here",
            responseObject = MyAPIResponse.class,
            since = "4.xx.yy",
            requestHasSensitiveInfo = false, responseHasSensitiveInfo = false,
            authorized = {RoleType.Admin, RoleType.ResourceAdmin, RoleType.DomainAdmin, RoleType.User})
public class MyAPICmd extends BaseCmd {
    public static final String APINAME = "myAPI";
```


## Challenges

Attempt and fix any upstream CloudStack API related issue(s):
https://github.com/apache/cloudstack/issues?q=is%3Aissue+is%3Aopen+label%3Aapi
