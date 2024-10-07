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
https://releases.hashicorp.com/vagrant/2.4.1/vagrant-2.4.1-1.x86_64.rpm
sudo apt install rpm
sudo rpm -i vagrant-2.4.1-1.x86_64.rpm
agrant version
Installed Version: 2.4.1
Latest Version: 2.4.1

You're running an up-to-date version of Vagrant!



### Vagrant disksize plugin

### Vagrant configuration 
vagrant reload


### Jenkins
Git authentification access 
jenkins user upgrade to sudoers users with NOPASSW
Add /bin/bash for the Jenkins shell configuration to execute 

### Anchore
https://github.com/anchore/anchore-cli
Entreprise Edition Workaround
