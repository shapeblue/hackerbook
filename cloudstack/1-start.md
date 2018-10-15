# Getting Started

- What is CloudStack: https://cloudstack.apache.org/about.html
- 2 Minute Introduction to Apache CloudStack: https://www.youtube.com/watch?v=oJ4b8HFmFTc
- Building IaaS with Apache CloudStack: https://www.youtube.com/watch?v=bxEL06BPGNw
- CloudStack - The best kept secret in the cloud: https://www.youtube.com/watch?v=nJiJRnJ_34w
- Building Clouds with Apache CloudStack: https://www.youtube.com/watch?v=4qFFwyK9hos

## Prerequisites

We assume you know:

- Basics of Virtualization (go try VirtualBox, VMware fusion, KVM etc)
- Basic Java, Python, shell scripting, git, maven
- Linux adminstration and usage

## Workstation Setup

Recommended laptop setup:
- Intel Quad core with VTx/VTd enabled
- 32GB RAM, 1TB SSD
- OS: Ubuntu 18.04+ (recommended), Fedora 25+

Good laptop models:
- Thinkpad P1 or P51/52s
- Thinkpad X1 Extreme
- Dell XPS 15

After fresh install, install following:

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

Python packages:

    pip install --upgrade cloudmonkey ansible pip

Install Chromium or Google Chrome:

    wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
    apt-get update
    apt-get install google-chrome-stable

Install manually: (optionals)

- Slack: https://slack.com/downloads/linux
- IntelliJ IDEA: https://www.jetbrains.com/idea/
- IntelliJ Goland: https://www.jetbrains.com/go/
- VS Code: https://code.visualstudio.com/

Recommended: Create and maintain a `dotfiles` repo for your env configuration.
For example, see https://github.com/rhtyd/dotfiles.

## Know the Project

- [The Apache Way](http://theapacheway.com/)
- [Project website](https://cloudstack.apache.org)
- [Join dev, users and other MLs](https://cloudstack.apache.org/mailing-lists.html)
- IRC channels on freenode: \#cloudstack, \#cloudstack-dev

Useful links:
- Git repo: https://github.com/apache/cloudstack
- Pull requests: https://github.com/apache/cloudstack/pulls
- Bug tracking: https://github.com/apache/cloudstack/issues
- Old bug tracking: https://issues.apache.org/jira/browse/CLOUDSTACK
- Docs: http://docs.cloudstack.apache.org/
- API docs: https://cloudstack.apache.org/api.html

Contributing Guideline:
https://github.com/apache/cloudstack/blob/master/CONTRIBUTING.md
