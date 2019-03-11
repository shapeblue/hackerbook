# Hypervisor

ServerResource
ServerResourceBase
VirtualRouterDeployer
CommandWrapper

## KVM

LibvirtComputingResource, KvmServerDiscoverer/LibvirtServerDiscoverer

## XenServer

CitrixResourceBase, XenServerResourceNewBase, XcpServerDiscoverer

## VMWare

VmwareResource
VmwareServerDiscoverer
VmwareHostService

How cloudstack orchestrates, programs hypervisor cmd-answers

TOs (transfer objects), service layer -> hypervisor (serverresource)

# Storage

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Storage+subsystem+2.0

StorageProcessor
StoragePoolResource
StorageSubsystemCommandHandler

## NFS and Local Storage

SecondaryStorageResource
LocalSecondaryStorageResource
NfsSecondaryStorageResource
PremiumSecondaryStorageResource
SecondaryStorageDiscoverer

Focus on NFS, how SSVM+NFS works, mount/unmount/copy/delete etc, use of apache2

## Ceph

# Case Study: Simulator

AgentRoutingResource
SimulatorDiscoverer
SimulatorSecondaryDiscoverer
