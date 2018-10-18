# Test Drive

The best way to learn about CloudStack is to start as a user, learn how to setup
and install it and test drive its features.

## Setting up CloudStack

Video: https://s3-eu-west-1.amazonaws.com/shapeblue-engineering-videos/hackerbook/1-user/1-user-task1.mp4

On your workstation, with KVM enabled and installed, using the
[virt-manager](https://virt-manager.org/) create a new VM
using [Ubuntu](https://www.ubuntu.com/download/server) 18.04+ ISO with at least
20GB disk, 2 CPUs and 4GB RAM. Before starting the VM, go to the VM's
setting and tick `Copy host CPU configuration`. Start the VM and complete the
installation.

Next, find out the IP of the VM, ensure that you're able to SSH into the host.
Ensure that you can SSH as `root` user with a known password. Also ensure
that the `universe` *apt* repository is enabled.

Next, we'll be building a local all-in-a-box cloud using CloudStack. You may
optionally refer to the CentOS based quick installation guide here:
http://docs.cloudstack.apache.org/en/latest/quickinstallationguide/qig.html

You'll use the Ubuntu 18.04 host you've created to install CloudStack. This
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
           gateway4: 192.168.122.1
           nameservers:
             addresses: [1.1.1.1,8.8.8.8]
           interfaces: [ens3]
           dhcp4: false
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

Note that now your VM should be accessible on the address 192.168.122.10. SSH
into it and install CloudStack management server and all other packages:

    apt-key adv --keyserver keys.gnupg.net --recv-keys 584DF93F
    echo deb http://packages.shapeblue.com/cloudstack/upstream/debian/4.11 / > /etc/apt/sources.list.d/cloudstack.list
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
the SSVM, CPVM, virtual routers etc. Seed the systemvm template on secondary
storage:

    wget http://packages.shapeblue.com/systemvmtemplate/4.11/systemvmtemplate-4.11.1-kvm.qcow2.bz2
    /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
              -m /export/secondary -f systemvmtemplate-4.11.1-kvm.qcow2.bz2 -h kvm \
              -o localhost -r cloud -d cloud

### KVM Setup

Enable VNC for console proxy:

    sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

Enable libvirtd in listen mode and configure non-TLS setup:

    sed -i -e 's/.*libvirtd_opts.*/libvirtd_opts="-l"/' /etc/default/libvirtd
    echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
    echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
    echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
    echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
    echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
    systemctl restart libvirtd

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

If you're deploying a zone using the onboarding wizard, click Launch or finalize
the setup and enable the zone. Watchout for any errors by tailing the management
server log.

**Recommended Exercise**:
- Deploy a Basic KVM based zone using the guided tour wizard
- CloudStack [Automation](hack/automation.md) using CloudMonkey and Ansible.

## Using your CloudStack IaaS

Congratulations, you've successfully installed and deployed a zone in your test
VM. Once your zone is launched or enabled, two system VMs - Console Proxy VM
(CPVM) and Secondary Storage VM (SSVM) will start which you can track in the UI
by going to Infrastructure > System VMs.

After the system VMs are `Up`, you can proceed to setup a template, deploy an
instance etc.
