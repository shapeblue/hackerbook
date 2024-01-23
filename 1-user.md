# Test Drive CloudStack

The best way to learn about CloudStack is to start as a user, learn how to setup
and install it and test drive its features.

* [Installing CloudStack](#installing-cloudstack)
    * [Validate your VM](#validate-your-vm)
    * [Configure Networking](#configure-networking)
    * [Install Packages](#install-packages)
    * [MySQL Server and CloudStack DB](#mysql-server-and-cloudstack-db)
    * [NFS Server](#nfs-server)
    * [SystemVM Template](#systemvm-template)
    * [KVM Setup](#kvm-setup)
    * [Security Configuration](#security-configuration)
    * [Setup Management Server](#setup-management-server)
* [Deploying Advanced Zone](#deploying-advanced-zone)
    * [Setup Zone](#setup-zone)
    * [Setup Network](#setup-network)
    * [Add Resources](#add-resources)
    * [Finishing Deployment](#finishing-deployment)
* [Using CloudStack](#using-cloudstack)
    * [Access](#access)
    * [CloudStack Feature Set](#cloudstack-feature-set)
    * [CloudStack Ops](#cloudstack-ops)

## Installing CloudStack

Video: https://s3-eu-west-1.amazonaws.com/shapeblue-engineering-videos/hackerbook/1-user/1-user-task1.mp4

**Recommended Reading**:
http://docs.cloudstack.apache.org/en/latest/installguide/

On your workstation, with KVM enabled and installed, using the
[virt-manager](https://virt-manager.org/) create a new VM
using [Ubuntu](https://www.ubuntu.com/download/server) 22.04 ISO with at least
30GB disk, 4 CPUs and 8GB RAM. Before starting the VM, go to the VM's
setting and tick `Copy host CPU configuration`. Start the VM and complete the
installation.

NOTE: DO NOT install or experiment anything from this chapter on your laptop,
but do them in a VM.

Next, find out the IP of the VM, ensure that you're able to SSH into the host.
Ensure that you can SSH as `root` user with a known password. Also ensure
that the `universe` *apt* repository is enabled.

Next, we'll be building a local all-in-a-box cloud using CloudStack. You may
optionally refer to the CentOS based quick installation guide here:
http://docs.cloudstack.apache.org/en/latest/quickinstallationguide/qig.html

You'll use the `Ubuntu 22.04` host you've created to install CloudStack. This
guide assumes that the VM is on a 192.168.122.0/24 network, can run KVM on it.

Reference:
https://rohityadav.cloud/blog/cloudstack-kvm/

### Validate your VM

SSH into your VM and make sure that hardware acceleration is available:

    cat /proc/cpuinfo| grep vmx | wc -l

All in a box setup:
- CloudStack Management server and Usage server
- MySQL server
- NFS server
- KVM hypervisor and CloudStack agent

Install basic packages:

    apt-get install openntpd vim htop bridge-utils

### Configure Networking

Create a file `/etc/netplan/01-netcfg.yaml` with following contents but remove
any yaml file at /etc/netplan: (change interface names, ip ranges accordingly)

     network:
       version: 2
       renderer: networkd
       ethernets:
         ens3:
           dhcp4: false
           dhcp6: false
           optional: true
       bridges:
         cloudbr0:
           addresses: [192.168.122.10/24]
           routes:
             - to: default
               via: 192.168.122.1
           nameservers:
             addresses: [1.1.1.1,8.8.8.8]
           interfaces: [ens3]
           dhcp4: true
           dhcp6: false
           parameters:
             stp: false
             forward-delay: 0

Save the file and apply network config, finally reboot:

    netplan generate
    netplan apply

Your connection will break since the VM's IP has changed, SSH again:

    ssh root@192.168.122.10

### Install Packages

Note that now your VM should be accessible on the address 192.168.122.10
(or IP of your VM). SSH into it and install CloudStack management server
and all other packages:

    mkdir -p /etc/apt/keyrings
    wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
    echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list
    apt-get update -y
    apt-get install cloudstack-management cloudstack-usage cloudstack-agent mysql-server nfs-kernel-server quota qemu-kvm

### MySQL Server and CloudStack DB

Make a note of the MySQL server's root user password. Configure InnoDB settings
in mysql server's `/etc/mysql/mysql.conf.d/mysqld.cnf` as follows:

    [mysqld]

    server_id = 1
    sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
    innodb_rollback_on_timeout=1
    innodb_lock_wait_timeout=600
    max_connections=1000
    log-bin=mysql-bin
    binlog-format = 'ROW'

Restart MySQL server and setup CloudStack database:

    systemctl restart mysql
    cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:<root password, default blank> -i 192.168.122.10

### NFS Server

Create NFS exports:

    mkdir -p /export/primary /export/secondary
    echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
    exportfs -a

Configure and restart NFS server:

    sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
    sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
    echo "NEED_STATD=yes" >> /etc/default/nfs-common
    sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
    service nfs-kernel-server restart

### SystemVM Template

CloudStack systemvm template is a special purpose guest template based on Debian
that provides a building block for creating service VMs in CloudStack such as
the SSVM, CPVM, virtual routers etc.

Note: From ACS 4.16 onwards, this is automatically seeded and you don't need to do anything.
This section has been added just for reference.

For 4.15 and older versions, something like the following are needed.

    wget http://packages.shapeblue.com/systemvmtemplate/4.15/systemvmtemplate-4.15.0-kvm.qcow2.bz2
    /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
              -m /export/secondary -f systemvmtemplate-4.15.0-kvm.qcow2.bz2 -h kvm \
              -o localhost -r cloud -d cloud

### KVM Setup

Enable VNC for console proxy:

    sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

Enable libvirtd in listen mode and configure non-TLS setup:

    sed -i -e 's/.*libvirtd_opts.*/libvirtd_opts="-l"/' /etc/default/libvirtd # For Ubuntu 18.04/20.04
    sed -i -e 's/^LIBVIRTD_ARGS=""/LIBVIRTD_ARGS="--listen"/' /etc/default/libvirtd # For Ubuntu 22.04
    echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
    echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
    echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
    echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
    echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
    systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket # For Ubuntu 20.04/22.04
    systemctl restart libvirtd
    
Note: while adding KVM host (default, via ssh) it may fail on newer distros which has OpenSSH version 7+ which has deprecated some legacy algorithms. This is only necessary for older ACS versions and you may not need to do this. To fix that the `sshd_config` on the KVM host may temporarily be changed to following before adding the KVM host in CloudStack:

    PubkeyAcceptedKeyTypes=+ssh-dss
    HostKeyAlgorithms=+ssh-dss
    KexAlgorithms=+diffie-hellman-group1-sha1

### Security Configuration

Configure firewall to allow port accessibility:

    # configure iptables
    NETWORK=192.168.122.0/24
    iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT
    iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT

    apt-get install iptables-persistent

    # Disable apparmour on libvirtd
    ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
    ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
    apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
    apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper

### Setup Management Server

Your installation and configuration is complete! You can now start the
management server as follows:

    cloudstack-setup-management

Open up http://192.168.122.10:8080/client in Chrome (recommended) or any
modern browser and log into the CloudStack UI using the username `admin` and
password `password.

To troubleshoot, check the management server logs for errors:

    tail -f /var/log/cloudstack/management/management-server.log

## Deploying Advanced Zone

Video: https://s3-eu-west-1.amazonaws.com/shapeblue-engineering-videos/hackerbook/1-user/1-user-task2.mp4

In this section, you'll create an advanced KVM-based zone in your CloudStack
environment. Skip the basic zone guided tour installation wizard, and proceed
to the `Infrastructure` Tab > Zone in the UI.

### Setup Zone

Click on add zone, select advanced zone and provide following configuration:

    Name - any name
    Public DNS 1 - 8.8.8.8
    Internal DNS1 - 192.168.122.1
    Hypervisor - KVM

### Setup Network

Use the default, which is `VLAN` isolation method on a single physical nic (on
the host) that will carry all traffic types (management, public, guest etc).

Public traffic configuration:

    Gateway - 192.168.122.1
    Netmask - 255.255.255.0
    VLAN/VNI - (leave blank)
    Start IP - 192.168.122.11
    End IP - 192.168.122.30

Pod Configuration:

    Name - any name
    Gateway - 192.168.122.1
    Start/end reserved system IPs - 192.168.122.31 - 192.168.122.50

Guest traffic:

    VLAN range: 100-200

### Add Resources

Create a cluster with following:

    Name - any name
    Hypervisor - Choose KVM

Add your default/first host:

    Hostname - 192.168.122.10
    Username - root
    Password - <password for root user>

Note: please check/enable ssh for `root` user in `sshd_config`

Add primary storage:

    Name - any name
    Scope - zone-wide
    Protocol - NFS
    Server - 192.168.122.10
    Path - /export/primary

Add secondary storage:

    Provider - NFS
    Name - any name
    Server - 192.168.122.10
    Path - /export/secondary

Next, click `Launch Zone` which will perform following actions:

    Create Zone
    Create Physical networks:
      - Add various traffic types to the physical network
      - Update and enable the physical network
      - Configure, enable and update various network provider and elements such as the virtual network element
    Create Pod
    Configure public traffic
    Configure guest traffic (vlan range for physical network)
    Create Cluster
    Add host
    Create primary storage (also mounts it on the KVM host)
    Create secondary storage
    Complete zone creation

Finally, confirm and enable the zone!

### Finishing Deployment

Wait for the system VMs to start before you can use your newly deployed zone.
You may troubleshoot using the management server logs for any errors.

**Recommended Exercise**:
- Deploy a Basic KVM based zone using the guided tour wizard

## Using CloudStack

Congratulations, you've successfully installed and deployed a zone in your test
VM. In this section, you'll learn CloudStack administration and usage.

Once your zone is enabled, before you proceed wait for two the system VMs to
come online - the Console Proxy VM (CPVM) and the Secondary Storage VM (SSVM).
You can track them in the UI at Infrastructure > System VMs. After they are up
you can use your CloudStack setup for testing various features and resource
lifecycles.

**Recommended Reading**:
http://docs.cloudstack.apache.org/en/latest/adminguide/

### Access

CloudStack has a query based HTTP API endpoint that can be used to access and
use the management server. There are two common modes of access:

- UI: The CloudStack UI is a VueJS and Ant Design. The UI is by default
  accessible at `http://<ip>:8080/client`.
- CLI: The CloudStack cloudmonkey (cmk) is the official CLI, it can be installed
  on Linux, Mac and Windows to access the API endpoint by default at
  `http://<ip>:8080/client/api`.

### CloudStack Feature Set

CloudStack feature sets can be broadly divided into three types:
- Business features
- Cloud (user) features
- Infrastructure (admin/management) features

#### Business

- Roles: Set of allow/reject APIs. The `Admin` and `User` are two widely used
  default roles.
- Accounts: Record of an individual user/customer or a team, all the resources
  in CloudStack are owned by an account. The `system` and `admin` are default
  CloudStack accounts. All accounts have a `role`.
- Users: A member/user of an account. Users in an account can be treated as
  aliases to the account. All accounts have one or more users.
- Domains: Accounts in a group. All account in a domain, by default in `/` the
  root domain.
- Authentication: Means of checking if you are who you say you are.
- Authorization: Means of checking if you can access resource `X` or have
  suitable privilege for other actions.
- LDAP and SAML: Two widely used authentication plugins in CloudStack.
- Projects: For organizing users and resources.
- Regions: Represents a CloudStack installation. Largely, an incomplete feature.
- Events: Used for auditing, monitoring, usage records and billing, and
  event-driven integration with other external systems.

#### Cloud

- Instance: Virtual machines.
- Volume: Disks of virtual machines.
- Volume snapshot: Checkpoint or snapshot of disk of a VM.
- VM snapshot: Checkpoint or snapshot of disk and memory of a VM.
- Template: Virtual machines disk that has a guest OS installed and can
  directly be used for creation of a VM.
- ISO: CDrom files for installation of a guest OS in a VM.
- Network:
  - Shared Network: a flat L3 network where VMs directly receive a public IP, a
    virtual router is usually deployed that provides DHCP and DNS services.
  - L2 Network: a flat L2 network with no services or virtual routers.
  - Isolated Network: a NAT-ed network that is provided by a virtual router that
    itself takes public IPs and provides a RFC1918 L3 private network with
    services to guest VMs like DHCP, DNS, NAT/SNAT, port-forwarding, firewall,
    vpn, load balancing etc.
  - VPC: similar to isolated network but provides multiple guest network tiers
    and ACLs for those network tiers.

#### Infrastructure

CloudStack has several organization units such as zones, pods, clusters, hosts
etc.

- Zone: Represents a datacenter or an availability zone
  - Basic zone: massively scalable AWS styled flat-network cloud usually with
    isolation and multitenancy implemented at host/hypervisor level by L2
    firewall (ebtables) rules by a feature called security groups.
  - Advanced zone: in additional to shared networks, provides enterprise
    network models with isolated and VPC networks with isolation provided by
    VLAN, VXLAN etc.
- Pod: Represents a rack.
- Cluster: Represents a group of hosts.
- Host: Hypervisor host that runs workloads/VMs. For example, KVM, VMware and
  XenServer.
- Storage:
  - Primary storage: Storage pool for virtual machine's disks.
    - Local storage: When disks are on same machine as the VM.
    - Shared storage: When disks are on a different machine, for example on a
      NFS server/host, Ceph, etc.
  - Secondary Storage: Image storage pool for templates, ISOs and snapshots.
- Physical network: Allows configuration of physical network, traffic labels,
  VLAN and IP address ranges for guest, public and private networks.
- System VMs: Special service VMs created and managed by CloudStack management
  server that implements an infrastructural service. They are based on a Debian
  based guest template called the `systemvmtemplates`. All system VMs run `sshd`
  on port `3922` and can be accessed as follows based on the hypervisor:
  - KVM and XenServer: ssh to the link-local IP of the systemvm on port `3922`
    using local `~/.ssh` key file.
  - VMware: ssh to the private IP of the systemvm from a management server host
    using the ssh key file at `/var/cloudstack/management/.ssh` location.

  Notable systemvm types:
  - SSVM: Secondary storage virtual machine provides means to manage the
    secondary storage, register/copy templates/ISOs, host snapshots and copy
    templates/ISOs to primary storage for consumption.
  - CPVM: Console proxy virtual machine provides means to access console of a
    VM. They act as a VNC/RDP proxy between the end user (browser) and the
    hypervisor where VMs run.
  - Virtual router: They provide router functionalities for various network
    models.
- Service offerings: Provides admins a way to create a catalogue of resource
  offerings to end users such as:
  - Compute
  - Disk
  - Network
  - System
- Limits and thresholds: Provides means for admins to define usage limits and
  threshold for various resource. Popular example is to set limit on
  accounts/domains and cpu/memory thresholds on clusters etc.

### CloudStack Ops

CloudStack management/usage/agent config directories and logs location:

| Service | Config | Logs |
| ------- | ------ | ---- |
| cloudstack-management | `dir:/etc/cloudstack/management/`, `file:/etc/default/cloudstack-management` | `dir:/var/log/cloudstack/management/` |
| cloudstack-usage | `dir:/etc/cloudstack/usage/`, `file:/etc/default/cloudstack-usage` | `dir:/var/log/cloudstack/usage/` |
| cloudstack-agent | `dir:/etc/cloudstack/agent/`, `file:/etc/default/cloudstack-agent` | `dir:/var/log/cloudstack/agent/` |

Go through each of the directories and files, notable files:
- `server.properties`: management server config file
- `db.properties`: database config file
- `agent.properties`: agent config file
- `log4j-cloud.xml`: log config file
- `key`: encryption password file

Troubleshooting references:
- http://docs.cloudstack.apache.org/en/latest/adminguide/troubleshooting.html
- https://www.slideshare.net/ShapeBlue/cloudstack-top-5-technical-issues-and-troubleshooting
- https://cwiki.apache.org/confluence/display/CLOUDSTACK/SSVM%2C+templates%2C+Secondary+storage+troubleshooting

**Recommended Exercises**:
- Repeat fresh installation and deployment with an EL (AlmaLinux, Rocky Linux or RHEL) environment https://www.youtube.com/watch?v=9gXEmWbgX2o
- Learn to read the CloudStack management server logs, `tail -f` the logs and
  deploy a fresh virtual machine with a new network, read and try to understand
  all the steps that happen during VM deployment.
- Attempt CloudStack [Automation](hack/automation.md) challenge using
  CloudMonkey and Ansible. (16 hours)
