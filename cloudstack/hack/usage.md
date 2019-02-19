# Usage Development

The CloudStack usage server is responsible for event processing, parsing and
creation of usage records for various usage types. The usage server has only
access to the `cloud_usage` database where it has several helper tables to
hold metadata about a resource, it's usage information and added/removed
timestamps.

Related references:
- http://docs.cloudstack.apache.org/en/4.11.2.0/adminguide/usage.html
- http://docs.cloudstack.apache.org/en/4.11.2.0/plugins/quota.html

The usage server can be started using maven using:

    mvn -P usage -Drun -Dpid=$$ -pl usage

CloudStack management server creates usage jobs in `cloud_usage.usage_job` table
based on the `usage.stats.job.aggregation.range` global setting. To create an
adhoc usage job, the API `generateUsageRecords` can be called.

Note:
- To enable debug logging, replace the `@USAGELOG@` to something like
  `usage.log` and `INFO` to `DEBUG` in
  `usage/target/transformed/log4j-cloud_usage.xml` and copy/rename it as
  `log4j.xml` to `usage/target/classes`.
- To run usage jobs quickly lower the usage aggregation interval.

In this exercise you'll learn how to add usage parsing and processing for a new
resource, `Coffee`. First define the usage type for the new resource in
`UsageTypes` and confirm it in the `listUsageTypes` response:

```java
    public static final int COFFEE = 30;
    // .. code redacted ..
    responseList.add(new UsageTypeResponse(COFFEE, "Coffee usage"));
```

Define a helper table such as `cloud_usage.usage_coffee` where you store the
`coffee_id`, `accountId`, `created` and `removed`, and any additional metadata
you want to export in the `listUsageRecords` response. Implement appropriate VO
and Dao classes in `engine/schema`. The `Dao` would at least require a method
to get usage records by accountId, start and end dates, and two helper methods
to save and remove usage record.

For example:

```sql
CREATE TABLE IF NOT EXISTS `cloud_usage`.`usage_coffee` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `account_id` bigint(20) unsigned NOT NULL,
  `domain_id` bigint(20) unsigned NOT NULL,
  `coffee_id` bigint(20) unsigned NOT NULL,
  `size` bigint(20) DEFAULT 0,
  `created` datetime NOT NULL,
  `removed` datetime,
  PRIMARY KEY (`id`),
  INDEX `i_usage_coffee` (`account_id`,`coffee_id`,`created`)
) ENGINE=InnoDB CHARSET=utf8;
```

From the API exercise, the create and remove APIs would emit create/remove
events that can be used by the usage server to parse and create usage records.

In `UsageManagerImpl`, write the parser and handlers such as:

```java
    @Inject
    private UsageCoffeeDao usageCoffeeDao;

    // In parseHelperTables:
    parsed = CoffeeUsageParser.parse(account, currentStartDate, currentEndDate);

    // In createHelperRecord:
    if (isCoffeeEvent(eventType)) {
        createCoffeeEvent(event);
    }

    // define isCoffeeEvent method
    private boolean isCoffeeEvent(String eventType) {
        return eventType != null && (eventType.equals(EventTypes.EVENT_COFFEE_CREATE) || eventType.equals(EventTypes.EVENT_COFFEE_DELETE));
    }

    // define createCoffeeEvent method
    private void createCoffeeEvent(final UsageEventVO event) {
        Long coffeeId = event.getResourceId();
        Long accountId = event.getAccountId();
        Date created = event.getCreateDate();
        Account account = _accountDao.findByIdIncludingRemoved(event.getAccountId());
         if (EventTypes.EVENT_COFFEE_CREATE.equals(event.getType())) {
            // Create UsageCoffeeVO and save it
        } else if (EventTypes.EVENT_COFFEE_DELETE.equals(event.getType())) {
            // Remove UsageCoffeeVO by the coffeeId
        }
     }
```

Note:
- The UsageCoffeeDao would need to only interact with the `cloud_usage` database.
- Transaction.execute can be used to ensure your db logic only gets executed
  against the `cloud_usage` db, see `QuotaUsageDaoImpl` as a modern example and
  `UsageVMInstanceDaoImpl` as an old-styled example.

Finally, you'll need to add a `CoffeeUsageParser` that will aggegrate coffee
usage by the account and persist usage record. For example:

```java
@Component
public class CoffeeUsageParser {
    private static UsageDao s_usageDao;
    private static UsageCoffeeDao s_usageCoffeeDao;

    @Inject
    private UsageDao usageDao;
    @Inject
    private UsageCoffeeDao usageCoffeeDao;

    @PostConstruct
    void init() {
        s_usageDao = usageDao;
        s_usageCoffeeDao = usageCoffeeDao;
    }

    public static boolean parse(AccountVO account, Date startDate, Date endDate) {
        if ((endDate == null) || endDate.after(new Date())) {
            endDate = new Date();
        }
        final List<UsageCoffeeVO> usageCoffees = s_usageCoffeeDao.getUsageRecords(account.getId(), startDate, endDate);
        if (usageCoffees == null || usageCoffees.isEmpty()) {
            LOGGER.debug("No Coffee usage for this period");
            return true;
        }
        for (final UsageCoffeeVO usageCoffee : usageCoffees) {
            // Logic to aggregate coffee usage by the account id
            // Use a hashmap for this
        }
        // Loop through the items in hashmap
        // Create UsageVO for aggregate coffee consumption per account
        final UsageVO usageRecord = new UsageVO(zoneId, account.getAccountId(),
                            account.getDomainId(), description, usageDisplay,
                            UsageTypes.COFFEE, rawUsage, null, null, null, null, null,
                            coffeeCount, null, startDate, endDate);
        s_usageDao.persist(usageRecord);
        return true;
    }
}
```

## Exercises

1. Implement the Coffee usage resource, its parser, helper tables, VO and Dao
   classes. To avoid maven/spring dependency hell, you may either implement a
   a new maven project `coffee-schema` and use that as dependency for your
   feature and the usage server, or simply add the schema related classes in the
   engine-schema project.

2. Validate that usage records are created by scheduled job or adhoc job created
   using `generateUsageRecords` API. Verify the usage records in the
   `listUsageRecords` response.

3. Verify that usage records stop generating when coffee is removed.
