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

## Prerequisites

We assume you know:

- Basic Linux adminstration and terminal usage
- Programming in Java, Python, shell scripting
- Development: git, maven3, IntelliJ IDEA, MySQL
- Tools: bash/zsh, ssh, scp, vi/emacs, wget/curl, tmux
- Basic virtualization (try KVM, VirtualBox, VMware fusion/workstation etc)

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

CloudStack architecture overview: https://www.youtube.com/watch?v=FLRtAzp_YuM

**Recommended reading**:
http://docs.cloudstack.apache.org/en/latest/conceptsandterminology/index.html

## Workstation Setup

Minimum laptop spec:
- Intel x64 i5 with HT and VTx/VTd enabled or equivalent AMD processor with AMD-V enabled
- 16GB RAM, 256GB hard disk
- OS: Ubuntu LTS (18.04/20.04) (recommended), Fedora 25+

Recommended laptop spec:
- Intel x64 i7/i9 with VTx/VTd enabled
- 32GB RAM, 1TB SSD
- OS: Ubuntu 20.04 (recommended)

Reference laptop models:
- Dell XPS 13/15 series
- HP Spectre or ZBook/Elite series
- Thinkpad P/X Extreme Series
- Any high spec gaming laptop

Laptop spec/build criteria:
- Does laptop meet development standard? Benchmark reference: https://browser.geekbench.com/v4/cpu/11716190
- Does laptop have at least 4 cores (>3Ghz/core), 16GB RAM, 512GB SSD/HDD?
- Does laptop have sturdy hinges, flex-resistant screen and keyboard?
- Does laptop have good input devices, ports, extensionabilty?

### Software Setup

Check if hardware virutalization is enabled on the workstation

    apt install cpu-checker
    kvm-ok

Setup your workstation with Ubuntu 22.04 and install following:

    sudo add-apt-repository universe
    sudo apt-get update
    sudo apt-get dist-upgrade
    # general packages
    sudo apt-get install vim git subversion mercurial patch rsync curl wget sed openssh-client gpg gnupg2 build-essential gzip bzip2 zip unzip p7zip-full p7zip-rar
    # cloudstack related development
    sudo apt-get install openjdk-11-jdk maven mysql-client mysql-server nfs-kernel-server quota genisoimage qemu-kvm qemu-utils libvirt-daemon virt-manager ipmitool jq uuid uuid-runtime python2 python2-dev python-setuptools python3-openssl python3-dev libffi-dev build-essential libssl-dev dpkg-dev libffi-dev rpm rpm2cpio bridge-utils iproute2 iptables ebtables ethtool vlan ipset tcpdump telnet fakeroot
    # security
    sudo apt-get install microcode.ctl intel-microcode amd64-microcode ca-certificates
    # opinionated development env (optional)
    sudo apt-get install zsh guake kazam ipython3 pv sshpass htop tmux tig vlc mutt bc cmake cmus cowsay gcc g++ wireshark openvpn network-manager-openvpn clisp

Note: If you're using Ubuntu 19.04+, [libmysql-java](https://packages.ubuntu.com/bionic/all/libmysql-java/filelist) package is missing and please manually install the latest mysql-connector-java manually at `/usr/share/java/` path. This is only needed when building/running older CloudStack versions.

In order to launch a Vm from the Virt-manager (Gui), Perform the following steps
    
    sudo systemctl enable --now libvirtd
    sudo systemctl start libvirtd
    sudo systemctl status libvirtd
    sudo usermod -aG kvm $USER
    sudo usermod -aG libvirt $USER

Logout and Logback so that the changes are applied 

    sudo virt-manager
  

Install pip and related packages on Ubuntu 22.04:

    curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
    sudo python2 get-pip.py
    sudo pip2 install --upgrade mysql-connector-python    

(Preferred) Install software using `snap`:

    sudo snap install slack --classic
    sudo snap install intellij-idea-community --classic
    sudo snap install vscode --classic
    sudo snap install pycharm-community --classic

Note: Intellij IDEA is the preferred and recommended IDE for developing CloudStack

(Optional) Gnome extensions:

    https://extensions.gnome.org/extension/1060/timezone/ (useful extension to track team around the world)
    https://extensions.gnome.org/extension/1497/topicons-redux/
    https://extensions.gnome.org/extension/120/system-monitor/
    https://extensions.gnome.org/extension/841/freon/
    https://extensions.gnome.org/extension/1082/cpufreq/

Productivity recommendations:
- Develop muscle memory for git, maven, vi/vim and IntelliJ IDEA.
- Create and maintain a `dotfiles` repo for your env configuration. For example,
  see https://github.com/rohityadavcloud/dotfiles.
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
