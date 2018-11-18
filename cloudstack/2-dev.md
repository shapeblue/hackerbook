# Basic CloudStack Development

## CloudStack Development 101

From an application setup and functioning perspective, a typical CloudStack
deployment consists of the following:

- CloudStack Management server(s)
- CloudStack Agent (on KVM and HyperV)
- CloudStack Usage server(s)

The CloudStack management server is a monolith Java application that embeds
the control plane, orchestrator and overall cloud controller.

The management server (sometimes written as mgmt server) has three types of
APIs:

- Platform API (REST-like or query-based API for end users and admins)
- Agent API (ServerResource based json/API that uses command-answer pattern
  handled by an implementation that can talk to a hardware resource)
- Plugin API (Set of Java interfaces and APIs to allow extension and
  modification of CloudStack)

The management server monolith has various layers:

- Presentation or API: the layer that implements and handles REST-like or query
  based APIs.
- Service/Business/Orchestration layer: the layer that usually handles API
  requests, manages business entities and participates in resource control.
- DB/Data access: the data access layer implements set of building blocks to talk
  to the database (MySQL).
- Kernel: Provides building blocks, polling, IPC, message bus, implements
  orchestration and controller for compute, network, storage, security etc.
- Agent/Cluster management: the layer that handles distributed system of mgmt
  server nodes and agents, and handles RPC mechanisms.

![](https://cwiki.apache.org/confluence/download/attachments/29687953/image2012-4-27%2012-43-50.png)

**Recommended Reading**:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Development+101
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/How+to+build+CloudStack
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+packages+and+dependencies
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Putting+CloudStack+together
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Data+Access+Layer
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Exceptions+and+logging
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Coding+conventions
- http://docs.cloudstack.apache.org/en/latest/developersguide

## Setting up Development Environment

The recommended development environment is Linux based, in this course Ubuntu
Linux 18.04+ is preferred. Run the following to install packages required for
CloudStack development on Ubuntu:

    $ sudo apt-get install openjdk-8-jdk maven python-mysql.connector libmysql-java mysql-server mysql-client bzip2 nfs-common uuid-runtime python-setuptools ipmitool genisoimage nfs-kernel-server quota

Older CloudStack versions may require older jdk/jre version, therefore setup,
install and learn to use `jenv`: http://www.jenv.be

The recommended setup is to run MySQL and NFS servers locally on your laptop
and the hypervisor in a VM or an external host. The CloudStack management and
usage server can be run using maven or via an IDE.

### Getting the source code

Apache CloudStack source code can be cloned from the following remotes:

- https://github.com/apache/cloudstack.git (Github, preferred)
- https://gitbox.apache.org/repos/asf/cloudstack.git (Gitbox, ASF hosted)

You may create a personal workspace and clone the repository, for example:

    mkdir -p ~/lab/
    cd ~/lab/
    git clone https://github.com/apache/cloudstack.git

The recommended directory structure may look something like:

```
        ~/lab/
        ├── cloudstack
        └── cloudmonkey
        └── monkeybox
        └── shapeblue
            └── ... private repositories ...
        └── ... other projects ...
```

Reference and reading resources:
- ProGit: https://git-scm.com/book/
- Try Git: https://try.github.io/
- Learn Git: https://www.codecademy.com/learn/learn-git

### Setup IDE

Setup IntelliJ IDEA (recommended) or any IDE of your choice. Get IntelliJ IDEA
community edition from:

    https://www.jetbrains.com/idea/download/#section=linux

Start the IDE, configure as needed and import CloudStack as:
- Click on import project
- Select the cloned `cloudstack` directory
- Select `Maven` as build system
- Select options suitably and import!

### Setup MySQL Server

After installing MySQL server, configure the following settings in its config
file such as at `/etc/mysql/mysql.conf.d/mysqld.cnf` and restart mysql-server:

    [mysqld]

    sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
    server_id = 1
    innodb_rollback_on_timeout=1
    innodb_lock_wait_timeout=600
    max_connections=1000
    log-bin=mysql-bin
    binlog-format = 'ROW'

### Setup NFS storage

After installing nfs server, configure the exports:

    echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
    mkdir -p /export/testing/primary /export/testing/secondary

Tip: You may want to have separate secondary storage for each version of
CloudStack. Rename and add more directories to the `/export/testing` path as and
when required.

The following is one way to seed a systemvmtemplate:

    wget http://packages.shapeblue.com/systemvmtemplate/4.11/systemvmtemplate-4.11.1-kvm.qcow2.bz2
    ./scripts/storage/secondary/cloud-install-sys-tmplt \
          -m /export/testing/secondary -f systemvmtemplate-4.11.1-kvm.qcow2.bz2
          -h kvm -o localhost -r cloud -d cloud

## Configure Environment

Put the following in your `sudoers` file using visudo etc. to allow processes
owned by your user (such as the CloudStack management server) to execute some
privileged commands:

    Cmnd_Alias CLOUDSTACK = /bin/mkdir, /bin/mount, /bin/umount, /bin/cp, /bin/chmod, /usr/bin/keytool, /bin/keytool

    Defaults:username !requiretty

    username ALL=(ALL) NOPASSWD:CLOUDSTACK

Tip: replace `username` with your Linux username.

## Building CloudStack

Noredist CloudStack builds requires additional jars that may be installed from:

    https://github.com/rhtyd/cloudstack-nonoss

To build CloudStack with `noredist` (this include vmware plugins etc):

    $ mvn clean install -Dnoredist -Dsimulator -P developer,systemvm

Deploy CloudStack database using:

    $ mvn -q -Pdeveloper -pl developer -Ddeploydb

Run management server using:

    $ mvn -pl :cloud-client-ui jetty:run  -Dnoredist -Djava.net.preferIPv4Stack=true

Install marvin:

    $ sudo pip install --upgrade tools/marvin/dist/Marvin*.tar.gz

To deploy an environment, you can use the deploy datacenter script as:

    $ python tools/marvin/marvin/deployDataCenter.py -i /path/to/config.cfg

When needed, the usage server can be started using:

    $ mvn -P usage -Drun -Dpid=$$ -pl usage

## Testing CloudStack

### Unit Testing

Unit tests in CloudStack are generally written with JUnit4 that also use
mockito, powermock and sometimes wiremock. You may learn more about JUnit4 and
usage of other libraries using existing unit tests (Ctrl+Shift+t in IntelliJ to
see an existing class's unit test) or by using following references:

- https://junit.org/junit4/faq.html
- http://www.vogella.com/tutorials/JUnit/article.html
- https://javacodehouse.com/blog/junit-tutorial

### Functional Testing

Functional or integration tests for CloudStack are written in Python 2.7 using
the Marvin test library. At the core of it, these are basically Python
`unittest` tests where the following test-probes are available:

- API client: run and get result of CloudStack APIs
- DB client: query MySQL database
- SSH client: allow remote access into a host endpoint/port

Typical integration tests are run using nose (a test runner), and an integration
test basically is a Python class that extends `cloudstackTestCase` which are
based on `unittest.case.TestCase`. The nature and mechanism of how these tests
run, make them integration or functional tests.

You may use following references to know more about using Python `unittest`
framework:

- https://docs.python.org/2/library/unittest.html
- http://pythontesting.net/framework/unittest/unittest-introduction
- https://www.geeksforgeeks.org/unit-testing-python-unittest

TODO: brief pointers on writing marvin based tests, using marvin utilities,
wrappers, building blocks etc.

## Simulator Based Development

CloudStack has a mocked hypervisor called `simulator` that may be used for
development of presentation, service, db and orchestration layer features.

You can use the `simulator` flag to build the simulator hypervisor plugin as:

    $ mvn clean install -Dsimulator -P developer,systemvm

Note in addition to deploying database the following must be run:

    $ mvn -q -Pdeveloper -pl developer -Ddeploydb-simulator

Run the management server using the `simulator` flag as well:

    $ mvn -pl :cloud-client-ui jetty:run -Djava.net.preferIPv4Stack=true -Dsimulator

Simulator based environment can be deployed using:

    $ python tools/marvin/marvin/deployDataCenter.py -i setup/dev/advanced.cfg

## MonkeyBox Based Development

Follow the MonkeyBox project for details:
https://github.com/rhtyd/monkeybox

## Debugging and Instrumentation

To debug any java process started by maven, you can export the following in
your shell (or include this by default in zshrc or bashrc):

    export MAVEN_OPTS="$MAVEN_OPTS -Xdebug -Xrunjdwp:transport=dt_socket,address=8787,server=y,suspend=n"

To remote-debug the KVM agent, put the following in
`/etc/default/cloudstack-agent` in your monkeybox and restart cloudstack-agent:

    JAVA=/usr/bin/java -Xdebug -Xrunjdwp:transport=dt_socket,address=8787,server=y,suspend=n

This will then allow you to attach a remote debugger on port `8787` (or any
other port you may have configured).

TODO:
- Using logs
- Debugging using IntelliJ
- Tools: Visual VM, Eclipse MAT

## CloudStack Packaging

CloudStack is typically packages and shipped as a deb or rpm repository. The
docker container images from https://hub.docker.com/u/bhaisaab can be used to
build CloudStack for CentOS (rpm) and Ubuntu (rpm).

The pre-requisite is that all the build and runtime dependencies are installed
on a build system (CentOS or Debian based) along with any nonoss dependencies.

Building on Debian/Ubuntu:

    # Option 1: Building using commands
    cd /root/of/the/cloudstack/repo
    dpkg-buildpackage -uc -us -b

    # Option 2: Building using script
    cd /root/of/the/cloudstack/repo
    bash -x packaging/build-deb.sh  # run with -h for help

Building on CentOS:

    # For el6/centos6
    cd /root/of/the/cloudstack/repo
    bash -x packaging/package.sh -p noredist -d centos63  # run with -h for help

    # For el7/centos7
    cd /root/of/the/cloudstack/repo
    bash -x packaging/package.sh -p noredist -o rhel7 -d centos7  # run with -h for help

## Contributing to CloudStack

For bug reporting create an issue: https://github.com/apache/cloudstack/issues

For any bugfix or improvement change(s) send a pull request: https://github.com/apache/cloudstack/pulls

For feature submission the typical process is as follows:

- Write a high level functional specification (FS).
- Start a discussion on dev@ mailing list with reference to the FS and/or any
  issues. Continue the discussion.
- Complete the feature, send a pull request (PR).
- Participate in code review, iterate implementation, request committers for
  review and merging. Typically every PR will be reviewed and (regression)
  tested. It is expected feature/bugfix PRs to have unit and integration tests.
- Send documentation PR.

**Case Study**: Dynamic Roles feature
- Functional specification: https://cwiki.apache.org/confluence/display/CLOUDSTACK/Dynamic+Role+Based+API+Access+Checker+for+CloudStack
- Mailing list: https://markmail.org/message/kkn5ihttg65i76kl
- Jira/bug ticket: https://issues.apache.org/jira/browse/CLOUDSTACK-8562
- Pull request and reviews: https://github.com/apache/cloudstack/pull/1489
- Documentation PR: https://github.com/apache/cloudstack-docs-admin/pull/37

## Basic Development Topics

| Topic | Effort |
| ----- | ------ |
| [Functional Spec](hack/spec.md) | 4-8 hours |
| [API Development](hack/api.md) | 8-24 hours |
| [Service Development](hack/service.md) | 8-16 hours |
| [DB Development](hack/db.md) | 8-16 hours |
| [UI Development](hack/ui.md) | 8-24 hours |
| [Pluggable Framework and Plugin development](hack/framework.md) | 8-16 hours |
| [IPC, Events and message bus](hack/ipc.md) | 4-8 hours |
| [RPC and Agent Framework](hack/rpc.md) | 4-8 hours |
| [Misc: Global Settings, Background Tasks](hack/misc.md) | 4-8 hours |
| [Functional Testing](hack/testing.md) | 8-16 hours |
