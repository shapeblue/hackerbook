# CloudStack DB

## Upgrade Paths

CloudStack has its own database framework that is based on
[DAO](https://www.oracle.com/technetwork/java/dataaccessobject-138824.html).

CloudStack uses MySQL as the database, any schema for any feature maybe define
in SQL. Before you start understanding and defining the DAO framework and its
usage in CloudStack, understand how CloudStack schema is defined and upgraded.
Use of terminal based `mysql-client` is recommended, you may also use [MySQL
workbench](https://dev.mysql.com/downloads/workbench/).

References:
- https://www.ibm.com/developerworks/java/library/j-genericdao/index.html
- https://www.w3schools.com/sql/
- https://www.tutorialspoint.com/sql/
- http://www.mysqltutorial.org/

CloudStack has following databases:
- `cloud`: the main database where most of CloudStack tables are
- `cloud_usage`: the database used by the usage server for usage record generation
- `simulator`: the database used by simulator plugin (only for developers, not
  for production usage)

The `DatabaseUpgradeChecker` class is responsible for CloudStack schema upgrade.
When the management server starts, an instance of this class kicks in to
check the version of the schema based on the `cloud.version` table against the
code version (from the jar). This class defines a map/sequence/hierarchy of
upgrade paths from a starting version number. The upgrade path is a class
that implements the `DbUpgrade` interface. For example, if the code version
(as defined in the root pom.xml file) is `4.12.0.0-SNAPSHOT` look for an upgrade
path class that may be named as `Upgrade4XXXXto41200`. This class would define
two sql scripts, one that upgrades CloudStack's schema and other that runs
to perform any schema cleanup. This class also defines metadata about the
upgrade path, the from/to version ranges etc. You may use any of the existing
upgrade paths to learn how to write one as well.

## Defining Schema

Pick the upgrade path for which you intend your feature for (i.e. the target
CloudStack version), and add relevant schema to that sql upgrade path file
(for example, `META-INF/db/schema-41120to41200.sql`). For example:

```sql
CREATE TABLE IF NOT EXISTS `cloud`.`coffee` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `uuid` varchar(40) UNIQUE,
  `name` varchar(255) NOT NULL,
  `description` varchar(1024) NULL,
  `state` varchar(40) NOT NULL,
  `account_id` bigint unsigned NOT NULL,
  `created` datetime NOT NULL COMMENT 'date of creation',
  `removed` datetime COMMENT 'date of removal',
  PRIMARY KEY (`id`),
  KEY (`uuid`),
  KEY `i_coffee` (`name`, `account_id`, `created`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Tips:
- Avoid using plural form for the table name (for example, `cloud.user` not
  `cloud.users`)
- Don't create schema keys that you won't need
- For performance gains use `views` instead of simply the `tables`, the general
  use case could be to speed up list API execution

## Writing DAO classes

**Recommended reading**: https://cwiki.apache.org/confluence/display/CLOUDSTACK/Data+Access+Layer

CloudStack data access layer is implemented by `GenericDaoBase` abstract class
that implements the DAO. For feature/resource specific tables, you would usually
implement a `Dao` interface that defines the contract for the `DaoImpl`, the
implementation class would extend `GenericDaoBase`.

The VO (view object) captures the schema/table and a VO instance typically
represents a row in the table. The VO typically implements the resource
interface (contract), however for purpose of any subsystem consuming a resource
object, the resource interface should be used/passed around than a VO instance.

The VO class exports and use several annotations for its table/db fields and
`@Table` to define the name of the table that the VO represents. Most CloudStack
tables have an internal (bigint) ID or database `id`, but the resources are
queries by users based on an external string based `uuid`.

Define the `VO` and make it implement the feature/resource interface. For
example:

```java
@Entity
@Table(name = "coffee")
public class CoffeeVO implements Coffee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private long id;

    @Column(name = "uuid")
    private String uuid;

    @Column(name = "name")
    private String name;

    @Column(name = "description")
    private String description;

    @Column(name = "state", nullable = false)
    @Enumerated(value = EnumType.STRING)
    private CoffeeState state = CoffeeState.Created;

    @Column(name = "account_id")
    private long accountId;

    @Column(name = GenericDao.CREATED_COLUMN)
    protected Date created;

    @Column(name = GenericDao.REMOVED_COLUMN)
    protected Date removed;

    // This empty constructor is needed for reflection to work
    public CoffeeVO() {
        uuid = UUID.randomUUID().toString();
    }

    // Custom constructor example
    public CoffeeVO(String name, long accountId) {
        this.uuid = UUID.randomUUID().toString();
        this.name = name;
        this.accountId = accountId;
    }

    // more custom constructors, getters, setters etc.
```

Define the `Dao` interface, for example:

```java
package org.apache.cloudstack.feature.dao;

public interface CoffeeDao extends GenericDao<CoffeeVO, Long> {
    // method definitions here
}
```

Define the `DaoImpl`, for example:

```java
package org.apache.cloudstack.feature.dao;

@Component
public class CoffeeDaoImpl extends GenericDaoBase<CoffeeVO, Long> implements CoffeeDao {
    // method implementations here
}
```

Declare the `DaoImpl` in the spring context xml file so that a bean gets created
and can be injected in the service layer class. For example:

```xml
--- a/plugins/hackerbook/feature/src/main/resources/META-INF/cloudstack/feature/spring-feature-context.xml
+++ b/plugins/hackerbook/feature/src/main/resources/META-INF/cloudstack/feature/spring-feature-context.xml
@@ -2,6 +2,9 @@
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                       http://www.springframework.org/schema/beans/spring-beans.xsd">
     <bean id="coffeeManager" class="org.apache.cloudstack.feature.CoffeeManagerImpl" />
+    <bean id="coffeeDaoImpl" class="org.apache.cloudstack.feature.dao.CoffeeDaoImpl" />
```

Now, the `DaoImpl` class can be injected and used by the service/manager class.
For example:

```java
public class CoffeeManagerImpl extends ManagerBase implements CoffeeManager, Configurable, PluggableService {
    // .. code redacted ..
    @Inject
    private CoffeeDao coffeeDao;
    // .. code redacted ..
    @Override
    public List<Coffee> listCoffees(ListCoffeesCmd cmd) {
        // Perform validations checks etc. following is just an example
        return new ArrayList<>(coffeeDao.listAll());
    }
    // .. code redacted ..
    @Override
    @ActionEvent(eventType = EventTypes.EVENT_COFFEE_CREATE, eventDescription = "creating coffee", async = true)
    public Coffee createCoffee(CreateCoffeeCmd cmd) {
        // Perform validations checks etc. following is just an example
        return coffeeDao.persist(new CoffeeVO(cmd.getName(), CallContext.current().getCallingAccountId()));
    }
```

By default, the Dao will have several ready to use methods such as `listAll`,
`findById`, `update`, `persist`, `remove` etc. When persisting a new VO, your
code does not need to create/set the `id`, `created` which are handled by the
DAO framework (GenericDaoBase).

Read the DAO wiki article to know various building blocks and utilities you can
use to create custom methods for searching and querying (for example, the
`SearchBuilder`).

## DB Transactions

**Recommended reading**: https://cwiki.apache.org/confluence/display/CLOUDSTACK/Database+Transactions

You can wrap a set of complex DB operations (for example, deleting of details
when coffee is deleted/removed) in a DB transaction using the `Transaction`
utility. For example:

```java
    return Transaction.execute(new TransactionCallback<CoffeeVO>() {
        @Override
        public CoffeeVO doInTransaction(TransactionStatus status) {
            return coffeeDao.persist(new CoffeeVO(name, description, accountId));
        }
    });
```

## FSM

TODO:

## Exercises

1. Implement the VO, Dao and DaoImpl classes for your feature, for each of the
   schema/tables `coffee` and `coffee_details`.

2. Integrate DB with the service layer, ensure all the APIs actually perform CRUD
   against the DB.

3. Write/update the unit tests (tip: use `mockito` to mock db interactions for
   the manager impl unit test, you can run a unit test using
   `@RunWith(MockitoJUnitRunner.class)`, and use `@Spy` and `@InjectMocks` on
   your service/manager impl class).

4. Implement FSM, use events for state transition and integrate it with a custom
   brewing simulation algorithm that introduces random sleep intervals. Finish
   the feature based on the spec.

Challenge: Attempt and fix any upstream CloudStack DB related issue(s):
https://github.com/apache/cloudstack/issues

## Details tables and scoped settings

List of `_details` tables in CloudStack:

    **account_details**
    autoscale_vmgroup_details
    autoscale_vmprofile_details
    **cluster_details**
    **data_center_details**
    disk_offering_details
    domain_details
    firewall_rule_details
    guest_os_details
    **host_details**
    image_store_details
    load_balancer_healthcheck_policy_details
    load_balancer_stickiness_policy_details
    network_acl_details
    network_acl_item_details
    network_details
    network_offering_details
    nic_details
    remote_access_vpn_details
    s2s_customer_gateway_details
    s2s_vpn_connection_details
    s2s_vpn_gateway_details
    service_offering_details
    snapshot_details
    snapshot_policy_details
    storage_pool_details
    usage_event_details
    user_details
    user_ip_address_details
    **user_vm_details**
    vlan_details
    vm_snapshot_details
    vm_template_details
    volume_details
    vpc_details
    vpc_gateway_details
