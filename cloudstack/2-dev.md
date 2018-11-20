# Basic CloudStack Development

* [CloudStack Development 101](#cloudstack-development-101)
* [Setting up Development Environment](#setting-up-development-environment)
    * [Getting the source code](#getting-the-source-code)
    * [Setup IDE](#setup-ide)
    * [Setup MySQL Server](#setup-mysql-server)
    * [Setup NFS storage](#setup-nfs-storage)
* [Configure Environment](#configure-environment)
* [Building CloudStack](#building-cloudstack)
* [Testing CloudStack](#testing-cloudstack)
    * [Unit Testing](#unit-testing)
    * [Functional Testing](#functional-testing)
* [Simulator Based Development](#simulator-based-development)
* [MonkeyBox Based Development](#monkeybox-based-development)
* [Debugging CloudStack](#debugging-cloudstack)
    * [Using Logs](#using-logs)
    * [Using IDE](#using-ide)
    * [Instrumentation](#instrumentation)
* [CloudStack Packaging](#cloudstack-packaging)
* [Contributing to CloudStack](#contributing-to-cloudstack)
* [Basic Development Topics](#basic-development-topics)

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

Create a Gitbub account in case you do not have one already: https://github.com

You may generate a public SSH key (if do not already have one at: `~/.ssh/id_rsa.pub`):
````
    ssh-keygen -t rsa -b 4096 -C "user_email@gmail.com"
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
````

Add your SSH key to you Github account:
- Go to your `Github` account
- Click on Settings and select SSH and GPG keys
- Click on `New SSH` key
- Add a title and paste the content of your key
- Click `Add SSH` key

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

You may configure IDEA per your preference or use this [settings
jar](https://github.com/rhtyd/dotfiles/tree/master/intellij) which you can import
in IDEA as `File > Import Settings > select jar file`.

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

Clone the above repository and execute the install script to install the noredist jar dependencies:

    $ cd /path/to/cloudstack-nonoss/repo
    $ bash -x install-non-oss.sh

To build CloudStack with `noredist` (this include vmware plugins etc):

    $ mvn clean install -Dnoredist -Dsimulator -P developer,systemvm

Deploy CloudStack database using:

    $ mvn -q -Pdeveloper -pl developer -Ddeploydb

Run management server using:

    $ mvn -pl :cloud-client-ui jetty:run -Dnoredist -Djava.net.preferIPv4Stack=true

Install marvin:

    $ sudo pip install --upgrade tools/marvin/dist/Marvin*.tar.gz

To deploy an environment, you can use the deploy datacenter script as:

    $ python tools/marvin/marvin/deployDataCenter.py -i /path/to/config.cfg

When needed, the usage server can be started using:

    $ mvn -P usage -Drun -Dpid=$$ -pl usage

Note: due to bug in `surefire` plugin that helps runs CloudStack's unit tests
you may need to use `-Djdk.net.URLClassPath.disableClassPathURLCheck=true`.

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

The [functional testing](hack/testing.md) exercise will cover in much detail
how to write marvin based integration tests and use the utilities, probes and
other building blocks of the Marvin library. As an example, you may look at the
[test/integration/smoke/test_dynamicroles.py](https://github.com/apache/cloudstack/blob/master/test/integration/smoke/test_dynamicroles.py) marvin test.

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

## Debugging CloudStack

To debug any java process started by maven, you can export the following in
your shell (or include this by default in zshrc or bashrc):

    export MAVEN_OPTS="$MAVEN_OPTS -Xdebug -Xrunjdwp:transport=dt_socket,address=8787,server=y,suspend=n"

To remote-debug the KVM agent, put the following in
`/etc/default/cloudstack-agent` in your monkeybox and restart cloudstack-agent:

    JAVA=/usr/bin/java -Xdebug -Xrunjdwp:transport=dt_socket,address=8787,server=y,suspend=n

This will then allow you to attach a remote debugger on port `8787` (or any
other port you may have configured).

### Using Logs

For a typical CloudStack installation, logs are found per service as follows:

- cloudstack-management: `dir:/var/log/cloudstack/management/`
- cloudstack-usage: `dir:/var/log/cloudstack/usage/`
- cloudstack-agent: `dir:/var/log/cloudstack/agent/`

However, when management server is launched using `maven` the logs will be in
the root directory of your source directory:
- vmops.log: the management server log
- api.log: the API log

Start the management server using `mvn` and try to read, follow and understand
what may be happening when the management server starts. For example, the first
thing you'll notice is that it:
- Loads module context (`spring-bootstrap-context.xml`) and creates a hierarchy
  of modules to load
- It starts loading modules in a certain order and starts integrity checks
- It configures various CloudStack components, instantiates registeries etc.
- Discovers API, starts various CloudStack components
- Finally the management server API service is available at port 8080 (default)
  and log statement like the following are seen:
```bash
2018-10-27 00:53:06,417 INFO  [c.c.c.ClusterManagerImpl] (Cluster-Heartbeat-1:ctx-4108c094) (logid:505420fb) We are good, no orphan management server msid in host table is found
2018-10-27 00:53:06,420 INFO  [c.c.c.ClusterManagerImpl] (Cluster-Heartbeat-1:ctx-4108c094) (logid:505420fb) No inactive management server node found
2018-10-27 00:53:06,437 DEBUG [c.c.c.ClusterManagerImpl] (Cluster-Heartbeat-1:ctx-4108c094) (logid:505420fb) Detected management node joined, id:1, nodeIP:127.0.0.1
```

Tip: Several CloudStack operations are scheduled and executed by its job
framework which gives such operations a unique job ID such as `job-123` and this
makes it easier to grep and investigate logs of a passed/failed job using the
management server logs of a multi-tenant (huge) CloudStack deployment.

**Challenge**: Deploy a VM (either using Simulator or KVM/monkeybox) and tail
through the logs and try to read and make sense of various steps that were
performed between the API request to deploy a VM was initiated and when the VM
came online.

### Using IDE

With the Java process (management server or agent) launched with above mentioned
flags, in IntelliJ IDEA go to `Run > Debug... > Edit configuration > Add >
Remote` and configure it suitably. Put breakpoint in code, start the debugger,
and wait for code execution such as an API request to reach the breakpoint to
debug.

**Challenge**: Add a breakpoint at the `execute()` of the `DeployVMCmdByAdmin`
class and step through the code to explore what gets executed when CloudStack
deploys a VM. Reference you findings against what you learnt by going through
the logs.

### Instrumentation

Several instrumentation and debugging tools exists that may be used to debug a
general Java/JVM application.

To list Java processes on a host:

    jps -l  # or ps aux | grep java

To get the thread dump of a process:

    jstack -l <pid>

To get the heap dump of a process:

    jmap -dump:format=b,file=heap.bin <pid>

Few popular instrumentation tools:

1. Visual VM

- Good tool for monitoring Java process CPU performance and memory, visualize
  threads, profile performance and memory usage, take and explore thread dumps,
  take and browse heap dumps and analyze core dumps.
- Download: https://visualvm.github.io/
- https://visualvm.github.io/documentation.html
- https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/

**Case study**: https://github.com/apache/cloudstack/pull/2314

2. Eclipse MAT

- Memory Analyzer Tool (MAT) is great heap analyzer and useful for finding
  memory leaks.
- Download: https://www.eclipse.org/mat

**Case study**: https://github.com/apache/cloudstack/pull/1729

Other notable mentions:
- Mission Control: https://www.oracle.com/technetwork/java/javaseproducts/mission-control/index.html
- JProfiler: https://www.ej-technologies.com/products/jprofiler/overview.html

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
