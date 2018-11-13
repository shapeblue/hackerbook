# Basic CloudStack Development

## CloudStack Development 101

TODO:
- High level Overview
- Architecture and Layers
- System Design

**Recommended Reference**:
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/Development+101
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/How+to+build+CloudStack
- http://docs.cloudstack.apache.org/en/latest/developersguide

## Setting up Development Environment

The recommended development environment is Linux based, in this course Ubuntu
Linux 18.04+ is preferred. Run the following to install packages required for
CloudStack development on Ubuntu:

    $ sudo apt-get install openjdk-8-jdk maven python-mysql.connector libmysql-java mysql-server mysql-client bzip2 nfs-common uuid-runtime python-setuptools ipmitool genisoimage nfs-kernel-server quota

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

        ~/lab/
        ├── cloudstack
        └── cloudmonkey
        └── monkeybox
        └── shapeblue
            └── ... private repositories ...
        └── ... other projects ...

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

- Unit tests
- Integration (Marvin) tests

## Configure Environment

Put the following in your `sudoers` file using visudo etc. to allow processes
owned by your user (such as the CloudStack management server) to execute some
privileged commands:

    Cmnd_Alias CLOUDSTACK = /bin/mkdir, /bin/mount, /bin/umount, /bin/cp, /bin/chmod, /usr/bin/keytool, /bin/keytool

    Defaults:username !requiretty

    username ALL=(ALL) NOPASSWD:CLOUDSTACK

Tip: replace `username` with your Linux username.

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

TODO: how to build local packages
https://hub.docker.com/u/bhaisaab/

## Contributing to CloudStack

Case study: Dynamic Roles
https://markmail.org/message/kkn5ihttg65i76kl

## Basic Development Topics

| Topic | Effort |
| ----- | ------ |
| [Functional Spec](hack/spec.md) | 4-8 hours |
| [API Development](hack/api.md) | 8-24 hours |
| [UI Development](hack/ui.md) | 8-24 hours |
| [Service Layer Development](hack/service.md) | 8-16 hours |
| [DB Layer Development](hack/db.md) | 8-16 hours |
| [Functional Testing](hack/testing.md) | 8-16 hours |
| [IPC, Events and message bus](hack/ipc.md) | 4-8 hours |
| [RPC and Agent Framework](hack/rpc.md) | 4-8 hours |
| [Misc: Global Settings, Background Tasks](hack/misc.md) | 4-8 hours |
| [Pluggable Framework and Plugin development](hack/framework.md) | 8-16 hours |
