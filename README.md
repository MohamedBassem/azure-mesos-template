# R-Console on a mesos cluster

A fork from : https://github.com/Azure/azure-quickstart-templates/tree/master/mesos-swarm-marathon

This Microsoft Azure template creates an Apache Mesos cluster running [R-Cluster](https://github.com/MohamedBassem/r-cluster) on a configurable number of machines.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMohamedBassem%2Fazure-rconsole-template%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Once your cluster has been created you will have a resource group containing 2 parts:

1. a masters machine in a master specific availability set. The master's SSH can be accessed via the public dns address at ports 2211

2. a set of agents behind in an agent specific availability set.  The agent VMs must be accessed through the master

All VMs are on the same private subnet, 10.0.0.0/18, and fully accessible to each other.

# R-Cluster

R-Cluster runs on the master machine and listens to port 5055. So it's accessible via "http://publicdnsaddress:5055". Mesos master runs on port 5050 and accessible via http://publicdnsaddress:5050.

# Installation Notes

Here are notes for troubleshooting:
 * the installation log for the masters, and agents are in /var/log/azure/cluster-bootstrap.log
 * event though the VMs finish quickly Mesos can take 5-15 minutes to install, check /var/log/azure/cluster-bootstrap.log for the completion status.

# Template Parameters
When you launch the installation of the cluster, you need to specify the following parameters:
* `newStorageAccountNamePrefix`: make sure this is a unique identifier. Azure Storage's accounts are global so make sure you use a prefix that is unique to your account otherwise there is a good change it will clash with names already in use.
* `adminUsername`: self-explanatory. This is the account used on all VMs in the cluster.
* `adminPassword`: self-explanatory
* `dnsNameForContainerServicePublicIP`: this is the public DNS name that you will use to connect to the cluster. You just need to specify an unique name, the FQDN will be created by adding the necessary subdomains based on where the cluster is going to be created. Ex. <userID>MesosCluster, Azure will add westus.cloudapp.azure.com to create the FQDN for the master.
* `agentCount`: the number of Mesos Agents that you want to create in the cluster
* `masterConfiguration`: You can specify if you want Masters to be Agents as well. This is a Mesos supported configuration otherwise Masters will not be used to run workloads.
* `agentVMSize`: The type of VM that you want to use for each node in the cluster. The default size is D1 (1 core 3.5GB RAM) but you can change that if you expect to run workloads that require more RAM or CPU resources.
* `masterVMSize`: Same as agentVMSize but for the master machine.
* `clusterPrefix`: this is the prefix that will be used to create all VM names. You can use the prefix to easily identify the machines that belongs to a specific cluster. If, for instance, prefix is 'c1', machines will be created as c1master1, c1agent1, c1agent5, ...
* `sshRSAPublicKey`: Configure all linux machines with the SSH rsa public key string.  Use 'disabled' to not configure access with SSH rsa public key.

# Common Tasks:

### SSH-ing to the master node
Assuming:
DNSname : mesos
Username : meso
Resource Group location was : westus

```bash
ssh mesos@mesos.westus.cloudapp.azure.com -p 2211
```

### SSH-ing to a certain agent

- SSH to master and then :
```
ssh 10.0.0.5[0-9]
```

# Questions

**Q.** My cluster just completed but Mesos is not up.

**A.** After your template finishes, your cluster is still running installation.  You can run "tail -f /var/log/azure/cluster-bootstrap.log" to verify the status has completed.
