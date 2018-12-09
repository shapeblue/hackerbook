# Testing

**Recommended reading list**:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Marvin+-+Testing+with+Python
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Automated+Tests+Rules+and+Guidelines

## Writing Marvin Test

A marvin based integration test is a Python `unittest` framework based class
that extends `cloudstackTestCase`, and run using the `nose` test runner. The
test can take input from the marvin (json) config file, the inbuilt-test
`test_data.py` map or any custom internal/external service map.

Here is an illustrated example on writing a Marvin based integration test:

```python
# Local and marvin related imports (tip: import specifics than all)
from marvin.cloudstackTestCase import cloudstackTestCase
from marvin.cloudstackAPI import (listCoffees)
from marvin.lib.utils import (cleanup_resources)
from marvin.lib.base import * # import all resources like Account
from marvin.lib.common import * # import all utility methods like get_zone etc.
from marvin.sshClient import SshClient

# Import runner related
from nose.plugins.attrib import attr

# Import system/additional libraries here
import os

class TestCoffee(cloudstackTestCase):

    # Define class globals here

    # Class's setUp method, runs only once per test class before test cases
    @classmethod
    def setUpClass(cls):
        testClient = super(TestCoffee, cls).getClsTestClient()
        cls.apiclient = testClient.getApiClient()
        cls.services = testClient.getParsedTestDataConfig()

        # Get attributes
        cls.zone = get_zone(cls.apiclient, testClient.getZoneForTests())
        cls.domain = get_domain(cls.apiclient)
        cls.hypervisor = testClient.getHypervisorInfo()

        # Define/create other resources
        cls.account = Account.create(
            cls.apiclient,
            cls.services["account"],
            domainid=cls.domain.id
        )
        cls.cleanup = [
            cls.account
        ]

    # Class's tearDown method, runs only once per test class after test cases
    @classmethod
    def tearDownClass(cls):
        try:
            cleanup_resources(cls.apiclient, cls.cleanup)
        except Exception as e:
            raise Exception("Warning: Exception during cleanup : %s" % e)

    # Define setUp that runs before each test case
    def setUp(self):
        # Define probes here
        # Probe: apiclient to make API calls
        self.apiclient = self.testClient.getApiClient()
        # Probe: DB client to query the database
        self.dbclient = self.testClient.getDbConnection()
        # Probe: SSH client to run remote commands
        self.sshclient = SshClient(
                              ipaddress,
                              22,  # port
                              username,
                              password,
                              retries=1,
                              log_lvl=logging.INFO
                         )
        # Get hypervisor detail
        self.hypervisor = self.testClient.getHypervisorInfo()
        # Command to get the default test_data config
        self.services = self.testClient.getParsedTestDataConfig()
        # List to hold any resources requiring cleanup
        self.cleanup = []

    # Define tearDown that runs after each test case
    def tearDown(self):
        try:
            cleanup_resources(self.apiclient, self.cleanup)
        except Exception as e:
            raise Exception("Warning: Exception during cleanup : %s" % e)

    # Define test cases with tags
    @attr(tags = ["advanced", "basic"], required_hardware="false")
    def test_list_coffee(self):
        # Use the inbuilt logger using `self.debug`
        self.debug("Test list coffee")

        # CloudStack build auto-generates API cmd/response python classes
        # All API command classes have the API name + `Cmd` suffix
        # Define the cmd object and its attributes
        cmd = listCoffees.listCoffeesCmd()
        # cmd.id = ...xyz...

        # Probe: Execute API by using `apiclient.<apiName>(cmd_obj)`:
        response = self.apiclient.listCoffees(cmd)

        # Probe: db example:
        row = self.dbclient.execute("select * from coffee where id=1")[0]

        # Probe: ssh client example:
        result = sshclient.runCommand("uname -a")

        # Tip: Add any resources created by test case to `self.cleanup.append(resource)`

        # Utility example: using wait_until to poll for a resource state/expectation
        def checkCoffeeBrewed():
            # Perform API call to list a coffee resource
            # Return true/false and any additional fields
            return coffee.state == "Brewed", None
        # Definition: wait_until(sleep_seconds, retries, callback, *callbackArgs)
        res, _ = wait_until(2, 30, checkCoffeeBrewed)
        if not res:
            self.fail("Failed to get Coffee in Brewed state")

        # Assertion examples:
        self.assertTrue(obj.attribute == conditional, "Some description")
        self.assertFalse(obj.attribute == conditional, "Some description")
        self.assertIsNotNone(obj, "Some description")
        self.assertEqual(
            isinstance(list_response, list),
            True,
            msg="Some message about response"
        )
        with self.assertRaises(Exception): # Assert failure on API
            self.apiclient.runSomeApi(cmd)

        # Skip/fail test examples:
        self.skipTest("Some message)
        self.fail("Some message")
```

## Exercises

1. Write test cases to verify positive/negative inputs for the CRUD APIs.

2. Create coffee and verify that the GC thread garbage collects it based on
   the `coffee` TTL global setting.

3. Verify brewing operation for addition plugins wherever possible.

Challenge: Attempt to fix any failing marvin integration test.
