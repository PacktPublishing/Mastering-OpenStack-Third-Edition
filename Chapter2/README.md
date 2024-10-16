# Chapter 2
Kicking Off the OpenStack Setup – The Right Way (DevSecOps)

## Description

The Chapter initiates the deployment of an OpenStack environment in different steps:
1. Test/Local environment: Running all components discussed in first chapter in a single machine
2. Create a private repository to host the infrastructure code for future collaboration and maintenance
3. Setup a deployer host running a CI/CD Tool - Jenkins 
4. Include a security stage check in the CI/CD pipeline to scan and test OpenStack services running in Kolla containers before proceeding the deployment in the target environment
5. CI/CD pipeline enabled to run based on approved/pushed code to the private repository and populate the changes to a target environment after passing through the all stages including security vulnerability check


## Code - How-To:

The Chapter uses the kolla-ansible community [repostority](https://github.com/openstack/kolla-ansible).

 Start by cloning the project code from Github:
```
git clone https://github.com/openstack/kolla-ansible.git
```

> [!TIP]
> To maintain own OpenStack infrastructure, create own private repository to reflect own custom configuration and desired infrastructure design state.


You can check the branch naming standard used by the OpenStack community in the Github page by clicking on the Switch branches/tags button the top right of the page:

![Branch Naming](IMG/Branches-Names-Standards.png)

Branches with **stable/** prefix are still maintained. Non maintained OpenStack releases are named with branches with **unmaintained/** prefix. 

> [!IMPORTANT]
> The cloned code is based on the master branch. OpenStack community keeps maintaining the kolla-ansible code of the latest 3 releases of OpenStack. It is recommended to use the master branch.  


## Deployment in Local environment:

To deploy OpenStack in a test local environment, a virtual environment can be installed in the local/dev machine with the following hardware and software pre-requesities:



## Troubleshooting:

### VirtualBox installation
### Vagrant installation
Install latest version  (2.4.1) 
wget https://releases.hashicorp.com/vagrant/2.4.1/vagrant-2.4.1-1.x86_64.rpm
sudo apt install rpm
sudo rpm -i vagrant-2.4.1-1.x86_64.rpm
vagrant version
Installed Version: 2.4.1
Latest Version: 2.4.1

You're running an up-to-date version of Vagrant!



### Vagrant disksize plugin

vagrant plugin install vagrant-disksize

### Vagrant configuration 
vagrant reload

### Shared local folder openstack_deploy
mkdir openstack_deploy

### Run Vagrant
vagrant up

Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'generic/ubuntu2204'...
Progress: 90%

## Make sure to install Docker
sudo apt-get install python3-dev libffi-dev gcc libssl-dev docker -y  
OR apt install docker.io




### Jenkins
Git authentification access 
jenkins user upgrade to sudoers users with NOPASSW
Add /bin/bash for the Jenkins shell configuration to execute 

### Anchore
https://github.com/anchore/anchore-cli
Entreprise Edition Workaround


## Nova user Guide: Create VM

1. Prepare image using Cirros image (https://docs.openstack.org/mitaka/install-guide-obs/glance-verify.html) (https://docs.openstack.org/image-guide/obtain-images.html#cirros-test)
 the login account is cirros. The password is gocubsgo.

$ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

2. Upload the image to Glance:

$ openstack image create 'Cirros' --file cirros-0.6.2-x86_64-disk.img --disk-format qcow2 --container-format bare --public
+------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                      |
+------------------+--------------------------------------------------------------------------------------------------------------------------------------------+
| container_format | bare                                                                                                                                       |
| created_at       | 2024-10-17T16:24:34Z                                                                                                                       |
| disk_format      | qcow2                                                                                                                                      |
| file             | /v2/images/a0198d59-6b58-4885-8aa8-cf41dba0a898/file                                                                                       |
| id               | a0198d59-6b58-4885-8aa8-cf41dba0a898                                                                                                       |
| min_disk         | 0                                                                                                                                          |
| min_ram          | 0                                                                                                                                          |
| name             | Cirros                                                                                                                                     |
| owner            | aa7e03d921cb4aec85e7c086abbfb99f                                                                                                           |
| properties       | os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/Cirros', owner_specified.openstack.sha256='' |
| protected        | False                                                                                                                                      |
| schema           | /v2/schemas/image                                                                                                                          |
| status           | queued                                                                                                                                     |
| tags             |                                                                                                                                            |
| updated_at       | 2024-10-17T16:24:34Z                                                                                                                       |
| visibility       | public                                                                                                                                     |
+------------------+--------------------------------------------------------------------------------------------------------------------------------------------+

3. Veriy the image:

$ openstack image list
+--------------------------------------+----------------------+--------+
| ID                                   | Name                 | Status |
+--------------------------------------+----------------------+--------+
| a0198d59-6b58-4885-8aa8-cf41dba0a898 | Cirros               | active |
| 9be4165a-45a7-4d55-b2ef-65ed8d243adc | manila-service-image | active |
+--------------------------------------+----------------------+--------+
