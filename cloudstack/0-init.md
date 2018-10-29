# Getting Started

Video: https://s3-eu-west-1.amazonaws.com/shapeblue-engineering-videos/hackerbook/0-init/0-init.mp4

* [About](#about)
* [Prerequisites](#prerequisites)
* [What is CloudStack?](#what-is-cloudstack)
* [Know the Project](#know-the-project)
* [Core Concepts](#core-concepts)
* [Workstation Setup](#workstation-setup)
* [References](#references-additional-books-and-readings--list)

## About

This fast learning course is designed to onboard new CloudStack developers
and get them to speed in few weeks by means of
[hacking](http://www.catb.org/esr/faqs/hacker-howto.html). Following guildelines
are recommended:

- Learn by programming, debugging and experimentation
- Avoid being stuck with only theory
- Do not attempt to understand everything the first time
- Do not read everything end to end
- Use Stackoverflow and Google for terms, concepts, programming questions
- Learn to ask the smart way: http://www.catb.org/esr/faqs/smart-questions.html

Authors:
- Rohit Yadav <rohit@apache.org>

Reviewers:
- None

## Prerequisites

We assume you know:

- Basic virtualization (try VirtualBox, VMware fusion, KVM etc)
- Programming in Java 8+, Python 2.7, shell scripting
- Development: git, maven3, IntelliJ IDEA, MySQL
- Tools: ssh, wget, curl, telnet, ping, traceroute, iptables, tail, vi
- Basic Linux adminstration and terminal usage

## What is CloudStack?

- Official answer: https://cloudstack.apache.org/about.html
- 2 minute video: https://www.youtube.com/watch?v=oJ4b8HFmFTc

Hour long talks: (optional)
- Building IaaS with Apache CloudStack: https://www.youtube.com/watch?v=bxEL06BPGNw
- Building Clouds with Apache CloudStack: https://www.youtube.com/watch?v=4qFFwyK9hos

## Know the Project

- [Project website](https://cloudstack.apache.org)
- [Join dev, users and other MLs](https://cloudstack.apache.org/mailing-lists.html)
- IRC channels on freenode: \#cloudstack, \#cloudstack-dev

Useful links:
- Docs: http://docs.cloudstack.apache.org/
- API docs: https://cloudstack.apache.org/api.html
- Git repo: https://github.com/apache/cloudstack
- Pull requests: https://github.com/apache/cloudstack/pulls
- Bug tracking: https://github.com/apache/cloudstack/issues
- Old bug tracking: https://issues.apache.org/jira/browse/CLOUDSTACK

Contribution guideline:
https://github.com/apache/cloudstack/blob/master/CONTRIBUTING.md

## Core Concepts

An IaaS (infrastructure as a service) platform provide means to partition and
primarily consume compute, storage and network resources usually by means of
virtualization.

Apache CloudStack is an IaaS platform with support for several hypervisors
such as KVM, VMware and XenServer. The usual setup consists of a management
server and the resoures it should manage such as the hypervisor and storage
hosts and networking configuration such as IP address ranges, VLANs etc.

The management server is a monolith that provides orchestration and control
plane accessible via query based http APIs through its UI and CLI. Various
authentication mechanisms such as the default authentication (pbkdf2), ldap,
saml2, api/secret key based etc are supported. Authorization is supported
through `Roles` which have access to a set/subset of APIs.

For user management CloudStack has `Domains` that have `Accounts` that have
`Users`, and all `Accounts` have some `Role`. It has `Projects` that allows
users across domains to participate as a team.

For cloud infrastructure management, CloudStack has concepts of organization
units much like filesystems have files and directories. Resources are in a
`Region` that represents a management server environment that can have `Zones`
that may be a datacenter. `Zone` can have `Pods` (like racks) that can have
`Clusters` that can have `Hosts` which runs your workload i.e. VMs. In addition,
there are `Primary Storage` (zone or cluster wide) that have the disks of an
instance and `Secondary Storage` (zone wide) that have disk templates, ISO
images and snapshots.

CloudStack supports many networking models and topologies:

- AWS-styled shared/flat network with L3 isolation (security groups)
- NAT-ed network with single (isolated network) and multiple tiers (VPC) with L2
  isolation such as VLANs with L3 services (nat, routing, firewall,
  port-forwarding, dhcp, dns etc) provided by a virtual router
- Pure L2 network with VLAN isolation

What is L2/L3, VLAN etc? We'll cover that in later chapters.

Lastly, CloudStack has several features including events and customizations via
compute, network, storage/disk, system offerings, and limits/thresholds/settings
for various resources.

**Recommended reading**:
http://docs.cloudstack.apache.org/en/latest/conceptsandterminology/index.html

## Workstation Setup

Recommended laptop setup:
- Intel x64 i7/i9 with VTx/VTd enabled
- 32GB RAM, 1TB SSD
- OS: Ubuntu 18.04+ (recommended), Fedora 25+

Minimum laptop setup:
- Intel x64 i5 with HT and VTx/VTd enabled or equivalent AMD processor with AMD-V enabled
- 16GB RAM, 256GB hard disk
- OS: Ubuntu 18.04+ (recommended), Fedora 25+

Good laptop models:
- Thinkpad P1 or P51/52s, X1 Carbon Extreme
- Dell XPS 15
- Alienware m15
- Any high spec gaming laptop

Setup your workstation with Ubuntu 18.04+ and install following:

    apt-get update
    apt-get dist-upgrade
    # general packages
    apt-get install vim git subversion mercurial patch rsync curl wget sed openssh-client gpg gnupg2 build-essential gzip bzip2 zip unzip p7zip-full p7zip-rar
    # cloudstack related development
    apt-get install openjdk-8-jdk maven mysql-client mysql-server libmysql-java nfs-kernel-server quota genisoimage qemu-kvm qemu-utils libvirt-bin virt-manager ipmitool jq uuid uuid-runtime python python-dev python-libvirt python-mysql.connector python-netaddr python-pip python-setuptools libssl-dev dpkg-dev libffi-dev rpm rpm2cpio bridge-utils iproute2 iptables ebtables ethtool vlan ipset tcpdump telnet fakeroot
    # security
    apt-get install microcode.ctl intel-microcode amd64-microcode ca-certificates
    # opinionated development env (optional)
    apt-get install zsh guake kazam ipython pv sshpass htop tmux tig vlc xchat irssi mutt bc cmake cmus cowsay dia gcc g++ wireshark openvpn network-manager-openvpn flashplugin-installer

Install python packages:

    pip install --upgrade cloudmonkey ansible pip

Install Chromium or Google Chrome:

    wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    apt-get update
    apt-get install google-chrome-stable

Install manually:

- IntelliJ IDEA: https://www.jetbrains.com/idea/ (recommended)
- IntelliJ Goland: https://www.jetbrains.com/go/
- VS Code: https://code.visualstudio.com/
- Slack: https://slack.com/downloads/linux

Productivity recommendations:
- Develop muscle memory for git, maven, vi/vim and IntelliJ IDEA.
- Create and maintain a `dotfiles` repo for your env configuration. For example,
  see https://github.com/rhtyd/dotfiles.
- Learn to touch type faster.

## References: Additional Books and Readings  List

CloudStack:
- [The Apache Way](http://theapacheway.com/)
- [Apache CloudStack bylaws](http://cloudstack.apache.org/bylaws.html)
- [Apache CloudStack docs](http://docs.cloudstack.apache.org)

Java:
- Effective Java
- Java Concurrency in Practice
- Java Performance
- Clean Code

Design and software engineering:
- The Pragmatic Programmer
- Release It!: Design and Deploy Production-Ready Software (Pragmatic Programmers)
- [SOLID papers](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)

Distributed systems:
- [Distributed Systems for fun and profit](http://book.mixu.net/distsys/)
- [Perspective on the CAP theorem](https://groups.csail.mit.edu/tds/papers/Gilbert/Brewer2.pdf) (paper)
- [Time, clocks and the ordering of events in a distributed system](https://amturing.acm.org/p558-lamport.pdf) (paper)
- [A comprehensive study of C/CRDTs](https://hal.inria.fr/file/index/docid/555588/filename/techreport.pdf) (paper)
