# Chapter 03
## OpenStack Control Plane - Shared Services

## Description

The Chapter initiates the deployment of an OpenStack environment for multi-node environment including:
-  1 x Cloud Controller Node
-  1 x Compute Node
-  1 x Network Node
-  1 x Storage Node



### System and Software Requirements:

To deploy OpenStack in a multi-node mode, the following hardware specifications are used:

| Hostname |vCPUs| RAM | Disk Space | Network Interfaces| Role 
|------|----|---------------|-------------|--------|--------|
| `cc01.os` |`8`| `128GB` | `250GB` | `4 x 10GB` | Cloud Controller| 
| `cn01.os` |`12`| `240GB` | `500GB` | `4 x 10GB` | Compute Node|  
| `net01.os` |`4`| `32GB` | `250GB` |`4 x 10GB` | Network Node| 
| `storage01.os` |`4`| `32GB` | `1TB` |`4 x 10GB` | Storage Node| 

> [!NOTE]
> The mentioned resources are being used in large production environments. Feel free to adjust the specs based on available resources you have but staying with minimum requirements to avoid performance issues. 


The chapter uses the different tools and software versions:

- **Operating System**: Ubuntu 22.04 LTS
- **kolla-ansible**: Latest and stable version from OpenStack Git master branch  (_Description in next section_)
- **Python**: Version 3.XX
- **Ansible Core**: Any version between ```2.16```  and ```2.17.99```.
- **Jenkins**: Any version for the latest Ubuntu/Debian Jenkins repository (_Description in next section_)



### Code - How-To:

The Chapter uses the kolla-ansible community [repostority](https://github.com/openstack/kolla-ansible).

Make sure you followed instructions to setup Jenkins and CI/CD pipeline for kolla-ansible deployment in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter02#3setting-up-the-cicd-pipeline)

Required files are:
- ```ansible/inventory/multi_packtpub_prod``` : Inventory file for multi-node setup
- ```etc/kolla/globals.yaml``` : OpenStack configurations and parameters 

You can check the branch naming standard used by the OpenStack community in the Github page by clicking on the Switch branches/tags button the top right of the page:

![Branch Naming](IMG/Branches-Names-Standards.png)

Branches with **stable/** prefix are still maintained. Non maintained OpenStack releases are named with branches with **unmaintained/** prefix. 



## Deployment of Multi-Node OpenStack environment:
### Example Production Topology: 

1. Hosts IP Allocation:

| Hostname |Role| Network Interface | Network Attachement | IP Address|  
|------|----|---------------|-------------|--------|
| `deployer.os.packtpub` |`Deployer`| `eth0` | `Management` | `10.0.0.15` | 
|            |             | `eth1` | `External` | `10.20.0.15` | 
| `cc01.os.packtpub` |`Cloud Controller`| `eth0` | `Management` | `10.0.0.100` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.100` | 
|            |             | `eth2` | `External` | `10.20.0.100` | 
|            |             | `eth3` | `Storage` | `10.30.0.100` | 
| `cn01.os.packtpub` |`Compute Node`| `eth0` | `Management` | `10.0.0.25` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.25` | 
|            |             | `eth2` | `External` | `10.20.0.25` | 
|            |             | `eth3` | `Storage` | `10.30.0.25` |  
| `net01.os.packtpub` |`Network Node`| `eth0` | `Management` |`10.0.0.30` |
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.30` | 
|            |             | `eth2` | `External` | `10.20.0.30` | 
|            |             | `eth3` | `Storage` | `10.30.0.30` | 
| `storage01.os.packtpub` |`Storage Node`| `eth0` | `Management` |`10.0.0.35` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.35` | 
|            |             | `eth2` | `External` | `10.20.0.35` | 
|            |             | `eth3` | `Storage` | `10.20.0.35` | 




### Deployment prepartion:

1. Configure on the Deployer node the hosts file with respective DNS entries where Ansible will run:

```sh
tee -a /etc/hosts<<EOF
### OPENSTACK SERVERS LIST
10.0.0.5 cc01.os
10.0.0.25 cn01.os
10.0.0.30 net01.os
10.0.0.35 storage01.os
EOF
```

2. Make sure to setup SSH keys so that the Deployer node can SSH password-less login to the OpenStack hosts for Ansible to run:

```sh
ssh-keygen
for i in cc01.os cn01.os net01.os storage01.os 
do 
  ssh-copy-id  -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@$i  
done
```

> [!NOTE]
> You can copy manually the generared `id_rsa.pub` file from the Deployer host to the OpenStack nodes located under `.ssh/authorized_keys`


3. Configure the hostnames and timezone for all OpennStack hosts:

```sh
for node in cc01.os cn01.0s net01.os storage01.os
do
  echo "=== Execute on $node ==="
  ssh root@$node hostnamectl set-hostname $node
  ssh root@$node timedatectl set-timezone Europe/Amsterdam
  echo ""
  sleep 2
done
```

4. For each OpenStack host, run an update and upgarde of the Ubuntu packages index:

```sh
apt-get update -y; apt-get upgrade -y
```

5. Install Docker engine in each OpenStack host:
 ```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### Deployment Configuration:
#### Assumptions:
-  Jenkins installed and running in the Deployer Node as explored in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline)
-  A local Docker registry is created as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#2-prepare-the-deployment-environment)



1. Create and copy the content of `/etc/kolla/globals.yaml` file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/etc/kolla/globals.yml). In this chapter the following settings in the `/etc/kolla/globals.yaml` file are used:

```sh
###################
# Ansible options
###################
kolla_base_distro: "ubuntu"
openstack_release: "master"
kolla_internal_vip_address: "10.0.0.47"
########################
# Nova - Compute Options
########################
nova_compute_virt_type: "kvm"
##############################
# Neutron - Networking Options
##############################
network_interface: "eth0"
neutron_external_interface: "eth1"
neutron_plugin_agent: "openvswitch"
###################
# OpenStack options
###################
enable_neutron_provider_networks: "yes"
################
# Docker options
################
docker_registry: 10.0.0.15:4000
docker_registry_insecure: "yes"
```

2. Create and copy the content of `/ansible/inventory/multi_packtpub` inventory file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/ansible/inventory/multi_packtpub_prod):

```sh
...
## Cloud Controller Node 
[control]
cc01.os.packtpub

## Compute Node 
[compute]
cn01.os.packtpub

## Network Node
[network]
net01.os.packtpub

## Storage Node
[storage]
storage01.os.packtpub

[deployment]
localhost       ansible_connection=local
...
```

3. Create a Jenkins Pipeline as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline). Follow the same instructions from ***Step 24*** . Use the Pipeline file in ***Step 26*** provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/Jenkinsfile).

```sh
..
PLAY RECAP ***************************************************************************************************************************************************
cc01.os.packtpub                  : ok=45   changed=0    unreachable=0    failed=0    skipped=43   rescued=0    ignored=0   
localhost                         : ok=31   changed=0    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0   
cn01.os.packtpub                  : ok=50   changed=0    unreachable=0    failed=0    skipped=48   rescued=0    ignored=0 
storage01.os.packtpub             : ok=35   changed=0    unreachable=0    failed=0    skipped=28   rescued=0    ignored=0 
net01.os.packtpub                 : ok=35   changed=0    unreachable=0    failed=0    skipped=28   rescued=0    ignored=0 
```

> [!IMPORTANT]
> Make sure to commit and push Jenkins pipelines files to your repository with the respective branch.
> e.g: `Jenkinsfile` in this chapter is pushed to a branch named `staging` as it targets different environment and not the same as described in `Chapter02`. 

