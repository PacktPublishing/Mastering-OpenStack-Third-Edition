# Chapter 03
## OpenStack Control Plane - Shared Services

## Description

The Chapter initiates the deployment of an OpenStack environment for multi-node environment including:
1. 1 x Cloud Controller Node
2. 1 x Compute Node
3. 1 x Network Node
4. 1 x Storage Node



### System and Software Requirements:

To deploy OpenStack in a multi-node mode, the following minimum hardware pre-requesities are used:

| Hostname |vCPUs| RAM | Disk Space | Network Interfaces| Role 
|------|----|---------------|-------------|--------|--------|
| `cc01.os` |`8`| `128GB` | `250GB` | `4 x 10GB` | Cloud Controller| 
| `cn01.os` |`12`| `240GB` | `500GB` | `4 x 10GB` | Compute Node|  
| `net01.os` |`4`| `32GB` | `250GB` |`4 x 10GB` | Network Node| 
| `storage01.os` |`4`| `32GB` | `1TB` |`4 x 10GB` | Storage Node| 




The chapter uses the different tools and software versions:

- **Operating System**: Ubuntu 22.04 LTS
- **kolla-ansible**: Latest and stable version from OpenStack Git master branch  (_Description in next section_)
- **Python**: Version 3.XX
- **Ansible Core**: Any version between ```2.16```  and ```2.17.99```.
- **Jenkins**: Any version for the latest Ubuntu/Debian Jenkins repository (_Description in next section_)



### Code - How-To:

The Chapter uses the kolla-ansible community [repostority](https://github.com/openstack/kolla-ansible).

Make sure you followed instructions to setup Jenkins and CI/CD pipeline for kolla-ansible deployment in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter02#3setting-up-the-cicd-pipeline)

Required files for Chapter03 are:
- ```ansible/inventory/multi_packtpub_prod``` : Inventory file for multi-node setup
- ```etc/kolla/globals.yaml``` : OpenStack configurations and parameters 

You can check the branch naming standard used by the OpenStack community in the Github page by clicking on the Switch branches/tags button the top right of the page:

![Branch Naming](IMG/Branches-Names-Standards.png)

Branches with **stable/** prefix are still maintained. Non maintained OpenStack releases are named with branches with **unmaintained/** prefix. 



## Deployment of Multi-Node OpenStack environment:
### Example Production Topology: 

1. The following topology is being deployed in Multi-Node OpenStack setup:

```
 
```


2. Hosts IP Allocation:

| Hostname |Role| Network Interface | Network Attachement | IP Address|  
|------|----|---------------|-------------|--------|
| `deployer.os` |`Deployer`| `eth0` | `Management` | `10.0.0.5` | 
|            |             | `eth1` | `External` | `10.20.0.5` | 
| `cc01.os` |`Cloud Controller`| `eth0` | `Management` | `10.0.0.15` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.15` | 
|            |             | `eth2` | `External` | `10.20.0.15` | 
|            |             | `eth3` | `Storage` | `10.30.0.15` | 
| `cn01.os` |`Compute Node`| `eth0` | `Management` | `10.0.0.25` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.25` | 
|            |             | `eth2` | `External` | `10.20.0.25` | 
|            |             | `eth3` | `Storage` | `10.30.0.25` |  
| `net01.os` |`Network Node`| `eth0` | `Management` |`10.0.0.30` |
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.30` | 
|            |             | `eth2` | `External` | `10.20.0.30` | 
|            |             | `eth3` | `Storage` | `10.30.0.30` | 
| `storage01.os` |`Storage Node`| `eth0` | `Management` |`10.0.0.35` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.35` | 
|            |             | `eth2` | `External` | `10.20.0.35` | 
|            |             | `eth3` | `Storage` | `10.20.0.35` | 




Management      10.0.0.0/24 100
Overlay/Tenant  10.10.0.0/24 200
External        10.20.0.0/24 300
Storage         10.30.0.0/24 400

2. Deployment prepartion:

On a multi-node setup, you will need to 

## Troubleshooting:

### Kolla Ansible

### Issue 0

### Issue 1

### Issue 2

### Issue 3: Create Cinder volume for vm disks 