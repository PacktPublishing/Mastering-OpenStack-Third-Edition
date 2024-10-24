# Chapter 3
## OpenStack Control Plane - Shared Services

## Description

The Chapter initiates the deployment of an OpenStack environment for multi-node environment including:
1. 1 x Cloud Controller Node
2. 1 x Compute Node
3. 1 x Network Node
4. 1 x Storage Node



## Code - How-To:

The Chapter uses the kolla-ansible community [repostority](https://github.com/openstack/kolla-ansible).

 Start by cloning the project code from Github:
```
git clone https://github.com/openstack/kolla-ansible.git
```

> [!TIP]
> Blank.


You can check the branch naming standard used by the OpenStack community in the Github page by clicking on the Switch branches/tags button the top right of the page:

![Branch Naming](IMG/Branches-Names-Standards.png)

Branches with **stable/** prefix are still maintained. Non maintained OpenStack releases are named with branches with **unmaintained/** prefix. 

> [!IMPORTANT]
> Blank


## Deployment in Local environment:

To deploy OpenStack in a Multi Node  environment, a virtual environment can be installed  with the following hardware and software pre-requesities:



## Troubleshooting:

### Kolla Ansible

### Jenkins
Git authentification access 
jenkins user upgrade to sudoers users with NOPASSW
Add /bin/bash for the Jenkins shell configuration to execute 

### Git
Custom repo based on local or remote Git Server.. Change the ***ci.os*** git server name/IP to use your server:
```
   stage('INSTALLING Kolla Ansible') {
      steps {
        echo '--INSTALLING Kolla Ansible --'
        sh '''#!/bin/bash 
        pip install git+https://git@ci.os/git/openstack_deploy/openstack_deploy
        kolla-ansible install-deps
        ''' 
      }
    } 
```

### Local Jenkins File kolla-ansible 

Jenkins Job to copy inventory and globals.yml files from private repository. The example uses the same directory structure based on default OpenStack folder structure

```
stage('Preparing Infrastructure') {
      steps {
        echo '--Preparing Infrastructure Files Structure --'
        sh ''' #!/bin/bash 
        sudo mkdir -p /etc/kolla
        sudo chown $USER:$USER /etc/kolla
        cp -r /usrlocal/share/kolla-ansible/etc_examples/kolla/* /etc/kolla 
        cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla 
        sed -i 's/^#kolla_base_distro:.*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
        sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/kolla/globals.yml
        sed -i 's/^#network_interface:.*/network_interface: "eth0"/g' /etc/kolla/globals.yml
        sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "eth1"/g' /etc/kolla/globals.yml
        sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "10.0.2.15"/g' /etc/kolla/globals.yml
        ''' 
      }
    }
```


### Create Cinder volume for vm disks 