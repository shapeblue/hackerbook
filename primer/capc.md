# CloudStack Cluster API Provider (CAPC)

## What is the Cluster API ?

[Kubernetes Cluster API](https://cluster-api.sigs.k8s.io/) is a Kubernetes project focused on providing declarative APIs and tooling to simplify provisioning, upgrading, and operating multiple Kubernetes clusters.

If this has too much jargon, it pretty much means that everyone has their own version of Kubernetes as a Service and the way they operate depends on the provider. CAPI aims to standardize and fix that!

CAPI defines the common CRUD operations on the Kubernetes Clusters along with default implementations for them. It aims to simplify the management of the cluster lifecycle and provide a single pane of glass to view and manage your Kubernetes clusters.

## How does it work ?

Have you seen inception ? If you have you'll have a pretty good idea! It's Kubernetes managing Kubernetes!

It starts off with running a Kubernetes cluster. It can run anywhere, on the cloud, in your infra or locally on your laptop!
This cluster is called your management cluster. It is this cluster that manages all your other Kubernetes Clusters.

With the `clusterctl` tool, you can initialize your cluster into a management cluster with the specific infrastructure provider.
This results in a bunch of pods running on your management cluster :
- CAPI Controller : This is the CAPI orchestrator that is the brain of the operation
- Control Plane Manager : This manages the control planes of your workload clusters and checks their status and if they are up and running
- Bootstrap Provider : This provides the relevant userdata which is used to convert your VM into a Kubernetes node
- Infrastructure Provider : This is the interface between CAPI and your underlying infrastructure. It defines Custom Resources that are specific to your target infrastructure. Eg: CloudStackCluster, CloudStakcMachine, etc

Now that you have a management cluster, it's time to create your workload clusters! It's these clusters that run on the specific infrastructure and can be exposed to the end-user.

Have a look at the presentation here: https://github.com/shapeblue/hackerbook/blob/main/primer/capi-capc.pdf

Also have a look at the documentation [here](https://cluster-api-cloudstack.sigs.k8s.io/) to get started

## CloudStack Cluster API Provider (CAPC)

With that introduction, let's move on to CAPC. CAPC is the CloudStack provider for CAPI. It provides an interface for CAPI to communicate with CloudStack

It consists of the following main components :
- `/api` : Contains the API definitions of the Custom resources used to create your cluster
- `/config` : Contains the yaml with replacements which later on becomes the CRDs
- `/controllers` : Contains the core of CAPC. It consists of the cluster reconciler and the machine reconciler. More on this later
- `/docs`: Contains the documentation for the project
- `/pkg` : Custom package to abstract the code used to communicate with CloudStack. Uses the cloudstack-go package
- `/tests` : Does pretty much what it says
- `/templates` : Contains the template YAMLs which are used as flavours while generating the workload cluster definition

Now that you've got a good overview, head on over to the official repo on how to get started! https://github.com/kubernetes-sigs/cluster-api-provider-cloudstack

## Internals

Coming to how it works, CAPI communicates with CloudStack to set up the workload cluster. It does so via `controllers`
There are two main files you need to care about. The `cloudstackcluster_controller.go` and `cloudstackmachine_controller.go`

### cloudstackcluster_controller :

This is file sets up all the resources required to bring up a cluster and cleans up the resources after a cluster has been deleted :
It in turn communicates with other reconcilers to create the necessary resources such as the networks, affinity groups, etc.

This functionality is handled by the `CloudStackClusterReconciler` in the `Reconcile` method. Based on whether the cluster is created or to be deleted, it takes the appropriate workflow. It runs initially when the cluster is created to set up the required resources. Only when it returns success, does it move on to setting up the nodes of the cluster.

### cloudstackmachine_controller :

This file deals with all the machines or VMs that become worker nodes. That includes :
- The VMs
- Adding control plane nodes to the Load Balancer rules
- Adding VMs to the Port Forwarding rules

This functionality is handled by the `CloudStackMachineReconciler` in the `Reconcile` method. Based on whether the machine is created or to be deleted, it takes the appropriate workflow. This runs periodically to ensure that the VM is up and a functional node. Based on how health checks are configured, it can delete a node if it is unhealthy and create a new one. It also brings up new nodes and removes old nodes in case of upgrades, scaling, etc.

### cloudstackaffinitygroup_controller :

This manages the required affinity groups for the cluster. Depending on the requirement, it either creates the affinity group, fetches it if already exists and deletes it when no longer required.

### cloudstackisolatednetwork_controller :

This manages the lifecycle of the isolated networks in ACS which power the CAPC cluster. Similar to other controllers, it creates or fetches (if network already exists) and deletes the network if created as part of the cluster.


### /api :

This directory contains the definitions of the Custom resources. These resources are CloudStack specific and used to internally keep track of the CloudStack resources mapped onto the CAPI definitions.

The main components are :

- CloudStackCluster : This is the definition of a CloudStack cluster. It contains the `CloudStackClusterSpec` which contains all the required information to get a cluster up and running such as the zone, network, endpoint, etc. The `CloudStackClusterStatus` keeps track of the CloudStack resources which are created and used in the workload cluster

- CloudStackMachine :  This is the definition of a CloudStack VM. It contains the `CloudStackMachineSpec` which contains all the required information to get a VM up and running such as the template, offering, ssh key, etc. The `CloudStackMachineStatus` keeps track of certain details of the VM used by CAPI

- Webhooks : These act as controls and run checks to ensure that the specs provided are valid. Examples can be ensuring that the zone, network, etc are not null and that they do not change during the lifecycle of the cluster


### Templates

CAPI requires that the VMs which become the Cluster nodes have all the necessary kuberntes binaries as well as the default images required baked into them. For creating these VM templates, we use the [image builder](https://github.com/kubernetes-sigs/image-builder) repo.
There are prebuilt images available [here](http://packages.shapeblue.com/cluster-api-provider-cloudstack/images/)

As of now CAPC supports RHEL 8, Rocky Linux 8 and Ubuntu 20.04 based images

## Exercise :

Follow the development steps mentioned [here](https://cluster-api-cloudstack.sigs.k8s.io/development/index.html) to get started with developing CAPC
Here are a simple exercises to get started :

1.  When a VM is created in CAPC, ensure that it is created with the suffix '-my-VM'.
    > This can get tricky as when the same VM needs to be queried from CAPC, it must match the name in ACS. Ensure that CAPC not only creates the VM with the suffix, but is also able to fetch it as well

    Hint : Go ahead and follow the logical flow of the `cloudstackmachine_controller` (and related code in `pkg/`) and identify where the VM is created and fetched and make the appropriate changes

2. Ensure that the offering specified has at least 2 Cores and 2048 MB.

    Hint : This can be checked when the offering is resolved.

    Bigger Hint : Follow the logical flow of the `cloudstackmachine_controller` (and related code `pkg/`) and identify where the offering is fetched and verify the details of the offering, and throw an error if the requirements are not met

## Documentation

- [Kubernetes Cluster API](https://cluster-api.sigs.k8s.io/)
- [Cluster API Provider CloudStack](https://cluster-api-cloudstack.sigs.k8s.io/)
