# Chapter 7
Running a Highly Available Cloud – Meeting the SLA


## Description

The Chapter extends and uses the deployment of a multi-node OpenStack environment as described in [Chapter03](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter03):
- Add HAProxy hosts 
- Add OpenStack Cloud Controller hosts
- Configure Keepalived and HAProxy 
- Add OpenStack network host
- Configure Neutron routing redundancy 
- Enable Masakari for OpenStack instances failover




<details close>
  <summary>Output</summary>

  ```sh

```

</details>


### System and Software Specs:

The chapter uses the same hardware and configuration for multi-node OpenStack setup as deployed in [Chapter03](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter03). The additional hosts in this chapter are having the following hardware specifications:


| Hostname |vCPUs| RAM | Disk Space | Network Interfaces| Role 
|------|----|---------------|-------------|--------|--------|
| `cc02.os` |`8`| `128GB` | `250GB` | `4 x 10GB` | Cloud Controller| 
| `cc03.os` |`8`| `128GB` | `250GB` | `4 x 10GB` | Cloud Controller| 
| `cn02.os` |`12`| `240GB` | `500GB` | `4 x 10GB` | Compute Node|  
| `net02.os` |`4`| `32GB` | `250GB` |`4 x 10GB` | Network Node| 
| `hap1.os` |`16`| `64GB` | `250GB` |`4 x 10GB` | Load Balancer Node| 
| `hap2.os` |`16`| `64GB` | `250GB` |`4 x 10GB` | Load Balancer Node| 

> [!NOTE]
> The mentioned resources are being used in large production environments. Feel free to adjust the specs based on available resources you have but staying with minimum requirements to avoid performance issues. 



The chapter uses the different tools and software versions:

- **Operating System**: Ubuntu 22.04 LTS
- **kolla-ansible**: Latest and stable version from OpenStack Git master branch  (_Description in next section_)
- **Python**: Version 3.X.X
- **Ansible Core**: Any version between ```2.16```  and ```2.17.99```.
- **Jenkins**: Any version for the latest Ubuntu/Debian Jenkins repository (_Description in next section_)



### Code - How-To:

The Chapter uses the kolla-ansible community [repostority](https://github.com/openstack/kolla-ansible).

Make sure you followed instructions to setup Jenkins and CI/CD pipeline for kolla-ansible deployment in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter02#3setting-up-the-cicd-pipeline)

Required files for Chapter03 are:
- ```ansible/inventory/multi_packtpub_prod``` : Inventory file for multi-node setup
- ```/etc/kolla/globals.yaml``` : OpenStack configurations and parameters 

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
| `hpa1.os.packtpub` |`Load Balancer`| `eth0` | `Management` | `10.0.0.20` | 
|            |             | `eth2` | `External` | `10.20.0.20` | 
| `hpa2.os.packtpub` |`Load Balancer`| `eth0` | `Management` | `10.0.0.21` | 
|            |             | `eth2` | `External` | `10.20.0.21` | 
| `cc02.os.packtpub` |`Cloud Controller`| `eth0` | `Management` | `10.0.0.101` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.101` | 
|            |             | `eth2` | `External` | `10.20.0.101` | 
|            |             | `eth3` | `Storage` | `10.30.0.101` | 
| `cc03.os.packtpub` |`Cloud Controller`| `eth0` | `Management` | `10.0.0.102` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.102` | 
|            |             | `eth2` | `External` | `10.20.0.102` | 
|            |             | `eth3` | `Storage` | `10.30.0.102` | 
| `cn02.os.packtpub` |`Compute Node`| `eth0` | `Management` | `10.0.0.26` | 
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.26` | 
|            |             | `eth2` | `External` | `10.20.0.26` | 
|            |             | `eth3` | `Storage` | `10.30.0.26` |  
| `net02.os.packtpub` |`Network Node`| `eth0` | `Management` |`10.0.0.31` |
|            |             | `eth1` | `Overlay/Tenant` | `10.10.0.31` | 
|            |             | `eth2` | `External` | `10.20.0.31` | 
|            |             | `eth3` | `Storage` | `10.30.0.31` | 


### Deployment prepartion:

1. Update on the Deployer node the hosts file with respective DNS entries of the additional Compute node:

```sh
tee -a /etc/hosts<<EOF
### First HAProxy Node
10.0.0.20 hpa1.os
### Second HAProxy Node
10.0.0.21 hpa2.os
...
### Second Cloud Controller Node
10.0.0.101 cc02.os
### Third Cloud Controller Node
10.0.0.102 cc03.os
...
### Second Compute Node
10.0.0.26 cn02.os
...
### Second Network Node
10.0.0.31 net02.os
...
EOF
```

2. Setup SSH keys so that the Deployer node can SSH password-less login to the additional Hosts:

```sh
ssh-copy-id -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@hpa1.os ; 
ssh-copy-id -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@hpa2.os ; 
ssh-copy-id -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@cc02.os ; 
ssh-copy-id -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@cc03.os ; 
ssh-copy-id -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@cn02.os ; 
ssh-copy-id -o StrictHostKeyChecking=no ~/.ssh/id_rsa.pub root@net02.os ; 
```

> [!NOTE]
> You can copy manually the generared `id_rsa.pub` file from the Deployer host to the OpenStack nodes located under `.ssh/authorized_keys`


3. Configure the hostnames and timezone for the additional hosts:

3. Configure the hostnames and timezone for the additional hosts:

```sh
for node in hpa1.os hpa2.0s cc02.os cc03.os cn02 net02
do
  echo "=== Execute on $node ==="
  ssh root@$node hostnamectl set-hostname $node
  ssh root@$node timedatectl set-timezone Europe/Amsterdam
  echo ""
  sleep 2
done
  ```

4. Run an update and upgarde of the Ubuntu packages index in each of the additional hosts:

```sh
apt-get update -y; apt-get upgrade -y
```

5. Install Docker engine in the additional hosts:
 ```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### Deployment Configuration:
#### Assumptions:
-  Jenkins installed and running in the Deployer Node as explored in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline)
-  A local Docker registry is created as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#2-prepare-the-deployment-environment)


1. Copy the `/ansible/inventory/multi_packtpub` inventory file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter07/ansible/inventory/multi_packtpub_prod) that includes the additional Compute node:

```sh
...
...

## Cloud Controller Hosts
[control]
cc01.os.packtpub
cc02.os.packtpub
cc03.os.packtpub


## Load Balancer Hosts
...
[haproxy]
hap1.os.packtpub
hap2.os.packtpub
...
[loadbalancer:children]
haproxy

....

[deployment]
localhost       ansible_connection=local
...
```

2. Create and copy the content of `/etc/kolla/globals.yaml` file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter07/etc/kolla/globals.yml). In this chapter the following settings in the `/etc/kolla/globals.yaml` file are used:

```sh
###################
# Ansible options
###################
kolla_base_distro: "ubuntu"
openstack_release: "master"
kolla_internal_vip_address: "10.0.0.47"
kolla_external_vip_address: "{{ kolla_internal_vip_address }}"
########################
# Nova - Compute Options
########################
nova_compute_virt_type: "kvm"
##############################
# Neutron - Networking Options
##############################
network_interface: "eth0"
neutron_external_interface: "eth2"
neutron_plugin_agent: "openvswitch"
###################
# OpenStack options
###################
enable_neutron_provider_networks: "yes"
enable_cinder_backend_lvm: "yes"
cinder_volume_group: "cinder-volumes"
enable_haproxy: "yes"
################
# Docker options
################
docker_registry: 10.0.0.15:4000
docker_registry_insecure: "yes"
```

3. Create a Jenkins Pipeline as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline). Follow the same instructions from ***Step 23*** . Use the Pipeline file in ***Step 25*** provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/Jenkinsfile).

```sh
..
PLAY RECAP ***************************************************************************************************************************************************
cc02.os.packtpub                  : ok=34   changed=0    unreachable=0    failed=0    skipped=23   rescued=0    ignored=0   
...
cc03.os.packtpub                  : ok=40   changed=0    unreachable=0    failed=0    skipped=23   rescued=0    ignored=0   
...
localhost                         : ok=12   changed=0    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0   
...
hap2.os.packtpub                  : ok=43   changed=0    unreachable=0    failed=0    skipped=76   rescued=0    ignored=0 
...
hap1.os.packtpub                  : ok=43   changed=0    unreachable=0    failed=0    skipped=76   rescued=0    ignored=0 
...
```

> [!IMPORTANT]
> Make sure to commit and push Jenkins pipelines files to your repository with the respective branch.
> e.g: `Jenkinsfile` in this chapter is pushed to a branch named `production` as it targets different environment and not the same as described in `Chapter02`. 


4. After the deployment is finished, check the new containers such as HAProxy and Keepalived in the respective nodes:

```sh
docker ps -a 
```

<details close>
  <summary>Output</summary>

  ```sh
CONTAINER ID     IMAGE                                                     COMMAND                     CREATED             STATUS                            PORTS     NAMES
...
2cda231524a1     registry/openstack.kolla/haproxy:master-rocky-9           "dumb-init--single-.."      40 minutes ago       Up 40 minutes ago (healthy)                haproxy
ca1b23d2c101     registry/openstack.kolla/keepalived:master-rocky-9        "dumb-init--single-.."      45 minutes ago       Up 45 minutes ago (healthy)              keepalived
...

```
</details>


5. Verify the content of the  `/etc/kolla/keepalived/keepalived.conf` file that should be contain an assigned `virtual_router_id` in the `vrrp_instance` snippet:

```sh
cat /etc/kolla/keepalived/keepalived.conf
```
<details close>
  <summary>Output</summary>

  ```sh
...
vrrp_instance kolla_internal_vip_51 {
  state BACKUP
  nopreempt
  interface br0
  virtual_router_id 51
  priority 40
  advert_int 1
  virtual_ipaddress {
    10.0.0.47 dev br0
    }
}
...

```
</details>


### Configuring routing redundancy with VRRP:
#### Assumptions:
-  Jenkins installed and running in the Deployer Node as explored in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline)
-  A local Docker registry is created as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#2-prepare-the-deployment-environment)


1. Copy/Edit the `/ansible/inventory/multi_packtpub` inventory file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter07/ansible/inventory/multi_packtpub_prod) that includes the additional Network node:

```sh 
...
## Network Nodes
[network]
net01.os.packtpub
###  Chapter 7 
## Adding a second Network Node 
net02.os.packtpub
...
```

2. Copy/Edit the `globals.yaml` file that includes additional setting for Neutron L3 agent HA:

```sh
enable_neutron_agent_ha: "yes"
```

3. Run the Jenkins Pipeline and make sure the new network node is properly installed:

```sh
..
PLAY RECAP ***************************************************************************************************************************************************

...
localhost                         : ok=12   changed=0    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0   
...
net02.os.packtpub                 : ok=34   changed=0    unreachable=0    failed=0    skipped=87   rescued=0    ignored=0 
...
```

4. Verify the Neutron L3 agent runs in both network nodes:

```sh
openstack network agent list --agent-type l3
```
<details close>
  <summary>Output</summary>

  ```sh

+--------------------------------------+--------------------+----------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host     | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+----------+-------------------+-------+-------+---------------------------+
| 52365341-ea3q-3121-dda1-76253daf3412 | L3 agent           | net01.os | None              | True  | UP    | neutron-l3-agent          |
| baba31aa-912d-5726-bbc1-761dca126f21 | L3 agent           | net02.os | None              | True  | UP    | neutron-l3-agent          |     
+--------------------------------------+--------------------+----------+-------------------+-------+-------+---------------------------+
```
</details>


### Configuring routing redundancy with DVR:
#### Assumptions:
-  Jenkins installed and running in the Deployer Node as explored in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline)
-  A local Docker registry is created as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#2-prepare-the-deployment-environment)


1. Copy/Edit the `/ansible/inventory/multi_packtpub` inventory file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter07/ansible/inventory/multi_packtpub_prod) that includes the additional Compute node:

```sh 
...
## Compute Nodes
[compute]
cn01.os.packtpub
###  Chapter 7 
## Adding a second Compute Node 
cn02.os.packtpub
...
[neutron-l3-agent:children]
compute
...
```

2. Copy/Edit the `globals.yaml` file that includes additional setting for enabling DVR routing capability:

```sh
enable_neutron_dvr: "yes"
```

3. Run the Jenkins Pipeline and make sure the new compute node is properly installed:

```sh
..
PLAY RECAP ***************************************************************************************************************************************************

...
localhost                         : ok=12   changed=0    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0   
...
cn02.os.packtpub                  : ok=54   changed=0    unreachable=0    failed=0    skipped=65   rescued=0    ignored=0 
...
```

4. Verify the Neutron L3 agent runs in both network nodes:

```sh
openstack compute service list --service nova-compute
```
<details close>
  <summary>Output</summary>

  ```sh

+--------------------------------------+--------------+-----------+------+----------+-------+----------------------------+
| ID                                   | Binary       | Host      | Zone | Status   | State | Updated At                 |
+--------------------------------------+--------------+-----------+------+----------+-------+----------------------------+
| 65a2da22-991a-72da-bca1-625daba1ef10 | nova-compute | cn01.os   | nova | enabled  | up    | 2024-10-20T18:51:08.000000 |
| 8db99811-ed1a-65da-9726-625da2310921 | nova-compute | cn02.os   | nova | enabled  | up    | 2024-10-29T21:49:06.000000 |
+--------------------------------------+--------------+-----------+------+----------+-------+----------------------------+
```
</details>




### Enable Instance Failover - Masakari

#### Assumptions:
-  Jenkins installed and running in the Deployer Node as explored in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#3setting-up-the-cicd-pipeline)
-  A local Docker registry is created as described in [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/README.md#2-prepare-the-deployment-environment)

1. Create and copy the content of `/etc/kolla/globals.yaml` file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/etc/kolla/globals.yml). In this part, the additional settings to deploy `Masakari` service in the `/etc/kolla/globals.yaml` file are used:

```sh
....
###################
# OpenStack options
###################
...
enable_masakari: "yes"
enable_hacluster: "yes"
...
```

2. Add the corresponding `magnum` services in `/ansible/inventory/multi_packtpub` inventory file if not assigned yet.

 `Masakari` `API`, `Engine`, `Hostmonitor` services will be running on the `Cloud Controller` nodes. `Masakari` `Instance Monitor` service will be running on `Compute` nodes. Additionally,  extra `hacluster` and `hacluster-remote` Ansible roles will be running on `Cloud Controller` nodes and   `Compute` nodes respectively. 
 
 The updated inventory file can be found [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter07/ansible/inventory/multi_packtpub_prod):

```sh
...
[masakari-api:children]
control
[masakari-engine:children]
control
[masakari-hostmonitor:children]
control
[hacluster:children]
control
[hacluster-remote:children]
compute
...
```


3.  Run the deployment using the  Jenkins Pipeline as described in [Chapter03](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/README.md#deployment-configuration). The Pipeline uses the stages provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter03/Jenkinsfile):

```sh
...
TASK [masakari : Creating Masakari database]**********************************************************************
...

TASK [masakari : Creating Masakari database user and setting permissions] ****************************************
...
TASK [masakari : Running Masakari bootstrap container]************************************************************
...
```

4. After the deployment is finished, check the new Masakari containers:

```sh
docker ps -a 
```

<details close>
  <summary>Output</summary>

  ```sh
CONTAINER ID     IMAGE                                                     COMMAND                     CREATED            STATUS                            PORTS     NAMES
dae2a88da21a     registry/openstack.kolla/masakari-engine:master-rocky-9   "dumb-init--single-.."      55 seconds ago     Up 15 seconds (health: starting)            masakari_engine
bbda3121b896     registry/openstack.kolla/masakari-api:master-rocky-9      "dumb-init--single-.."      22 seconds ago     Up 2 seconds (health: starting)             masakari_api
...

```
</details>


5. Once Masakari containers are up and running, verify the service endpoint is created:

```sh
openstack service list
```
<details close>
  <summary>Output</summary>

 ![ServiceList](IMG/masakari-service-list.png)
</details>





## Troubleshooting:





