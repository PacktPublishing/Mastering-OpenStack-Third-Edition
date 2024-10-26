# Chapter 02
## Kicking Off the OpenStack Setup – The Right Way (DevSecOps)

## Description

The Chapter initiates the deployment of an OpenStack environment in different steps:
1. Test/Local environment: Running all components discussed in first chapter in a single machine for testing using Vagrant and VirtualBox.
2. Setup a deployer host running a CI/CD Tool using Jenkins 
3. Create CI/CD pipeline for the OpenStack code deployment. 
4. Include a security stage check in the CI/CD pipeline to scan and test OpenStack services running in Kolla containers before proceeding the deployment in the target environment
6. **Bonus Section**: Troubleshooting of most encoutred issues when running an OpenStack deployment.  



## System and Software Requirements:

To deploy OpenStack in a test/local environment, an environment can be installed in a physical or virtual machine with the following minimum hardware and software pre-requesities:

- **Operating System**: Ubuntu 22.04 LTS
- **CPU**: Minimum of multi-core AMD64 processor with 4 cores
- **RAM**: Minimum of 8GB RAM
- **Disk space**: Minimum of 50 GB free storage (root Disk)

The chapter uses the different tools and software versions:
- **kolla-ansible**: Latest and stable version from OpenStack Git master branch  (_Description in next section_)
- **Python**: Version 3.XX
- **Ansible Core**: Any version between ```2.16```  and ```2.17.99```.
- **Vagrant**: Latest Version  ```2.4.1``` (_At the time of writing this edition_).
- **Jenkins**: Any version for the latest Ubuntu/Debian Jenkins repository (_Description in next section_)


### Code - How-To:

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

## 1. Deployment in a Local Environment:

1. Update the package lists from the repositories of Ubuntu OS:
```sh
$ sudo apt-get update
```

2. Install ```Virtualbox``` in the local machine:
```sh
$ sudo apt install -y virtualbox virtualbox-ext-pack
```

3. Install the Vagrant version ```2.4.1```:
```sh
$ wget https://releases.hashicorp.com/vagrant/2.4.1/vagrant-2.4.1-1.x86_64.rpm
$ sudo apt install rpm
$ sudo rpm -i vagrant-2.4.1-1.x86_64.rpm
```

4. Check the installed version of ```vagrant```:
```sh
$ vagrant version
```

<details close>
  <summary>Output</summary>

  ```sh
Installed Version: 2.4.1
Latest Version: 2.4.1
You're running an up-to-date version of Vagrant!
```
</details>

5. Install ```disksize``` plugin for Vagrant to define disk medium to Vagrant host guests and allow disk resize:

```sh
$ vagrant plugin install vagrant-disksize
```

6. Create optionally a shared directory for Vagrant guest hosts:

```sh
$ mkdir openstack_deploy
```

7. Create and copy the content of the Vagrant file named ```Vagrantfile``` provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter2/Vagrantfile).
<br />

- ```config.vm.box``` : Install Ubuntu 22.04 TLS 
- ```config.disksize.size```: Setup root disk size of 50GB 
- ```config.vm.synced_folder```: Sync directory with local folder ```openstack_deploy```
- ```config.vm.network```: Create 2 interfaces ```eth0``` and ```eth1``` with port forwarding on ports 80 and 8080.
- ```vb.memory```: Set RAM size of 8GB 
- ```vb.cpus```: Set CPU cores of 4 

8. Save the file and run the deployment of the Vagrant guest machine:
```sh
$ vagrant up
```

<details close>
  <summary>Output</summary>

  ```sh
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'generic/ubuntu2204'...
Progress: 90%
```
</details>

9. SSH the Vagrant guest from the local machine (_username/password are NOT required_):

```sh
$ vagrant ssh
```



10. Once logged in, install the following packages and dependencies:

```sh
sudo -i
apt update -y
apt install python3-dev libffi-dev gcc libssl-dev python3-venv
apt install python3-pip
```

11. Create optionally a Python virtual envionment:

```sh
python3 -m venv local
source local/bin/activate
```

12. Install ```pip```:

```sh
(local)# pip install -U pip
```


13. Install Ansible core:
```sh
(local)# pip install 'ansible-core>=2.16,<2.17.99'
```

14. Install kolla-ansible:
```sh
(local)# pip install git+https://opendev.org/openstack/kolla-ansible@master
(local)# kolla-ansible install-deps
```



15. Create a new directory locally to prepare for the kolla-ansible run:
```sh
(local)# mkdir -p /etc/kolla
(local)# chown $USER:$USER /etc/kolla
```

16. Step ```14``` should have downloaded example files of kolla-ansibe repository that should be located under ```/local/share/kolla-ansible/```. Copy skeleton files for both `all-in-one` inventory and ```globals.yaml``` files under ```/etc/kolla```directory:

```sh
(local)# cp -r local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla 
(local)# cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla 
```

17. Create and copy the content of ```/etc/kolla/globals.yaml``` file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/etc/kolla/globals.yml).
You can also run the following command lines to get the same settings in the ```/etc/kolla/globals.yaml``` file  :
<br />

```sh
sed -i 's/^#kolla_base_distro:.ls*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/kolla/globals.yml
sed -i 's/^#network_interface:.*/network_interface: "eth0"/g' /etc/kolla/globals.yml
sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "eth1"/g' /etc/kolla/globals.yml
sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "10.0.2.15"/g' /etc/kolla/globals.yml
```

18. Generate the OpenStack services secretes:

```sh
kolla-genpwd -p /etc/kolla/passwords.yml
```



19. Bootstrap the OpenStack services in ```all-in-one``` configuration:
```sh
kolla-ansible -i /etc/kolla/all-in-one bootstrap-servers
```
<details close>
  <summary>Output</summary>

![Bootstrap](IMG/kolla-ansible-bootstraps.png)

</details>

20. Run The Pre-checks script in ```all-in-one``` configuration:
```sh
kolla-ansible -i /etc/kolla/all-in-one prechecks
```

<details close>
  <summary>Output</summary>

![PRECHECKS](IMG/kolla-ansible-prechecks.png)

</details>

21. Run the OpenStack services deployment in ```all-in-one``` configuration:

```sh
kolla-ansible -i /etc/kolla/all-in-one deploy
```
<details close>
  <summary>Output</summary>

  ![PRECHECKS](IMG/kolla-ansible-deploy.png)

</details>

22. Verify that all containers are up and running and in ```healthy``` status:

```sh
docker ps -a
```

![List Container Services](IMG/docker-ps-a-all.png)


23. Run post deploy script to generate cloud configuration file:
```sh
kolla-ansible -i /etc/kolla/all-in-one post-deploy
```

24. Check the config file generated under ```/etc/kolla/``` named ```clouds.yaml```:

```sh
cat /etc/kolla/clouds.yaml
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

25. You can also source the generated file ```admin_openrc.sh``` to start interacting with OpenStack APIs:
```
# source /etc/kolla/admin_openrc.sh
```
26. To interact with the dashboard for ```admin``` user, capture the password from either the  ```clouds.yaml```,  ```admin_openrc.sh``` or simply from ```passwords.yml``` file:

```sh
grep keystone_admin_password /etc/kolla/passwords.yml
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

27. Open a browser on local host and navigate to the VIP address configured in the ```globals.yml``` file:

![List Container Services](IMG/WelcomeOS.png)
![List Container Services](IMG/Main-Dashboard-UI2.png)


28. To interact with OpenStack services using CLI, install the OpenStack client CLI tools:

```sh
pip install python-openstackclient
```

29. Make sure the variables in the ```admin-openrc.sh``` are sourced and verify the OpenStack CLI:
``` sh
openstack services list
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

## 2. Prepare The Deployment Environment:
### 2.1. Create Own Repository:
### 2.2. Optional - Create Own Private Container Registry:

1. Install ```kolla``` command line tool:
```sh
$ sudo python3 -m pip install kolla
```
<details close>
  <summary>Output</summary>

  ```sh
Collecting kolla
  Downloading kolla-18.2.0-py3-none-any.whl (356 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 356.4/356.4 KB 1.8 MB/s eta 0:00:00
Requirement already satisfied: Jinja2>=3.0.1 in /usr/local/lib/python3.10/dist-packages (from kolla) (3.1.4)
Requirement already satisfied: oslo.config>=5.1.0 in /usr/local/lib/python3.10/dist-packages (from kolla) (9.6.0)
Requirement already satisfied: pbr!=2.1.0,>=2.0.0 in /usr/local/lib/python3.10/dist-packages (from kolla) (6.1.0)
Collecting GitPython>=1.0.1
  Downloading GitPython-3.1.43-py3-none-any.whl (207 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 207.3/207.3 KB 1.8 MB/s eta 0:00:00
     ....
```
</details>

2. Create a local Docker registry named ```master_registry```:
```sh
$ docker run -d  --network host --name master_registry --restart=always -e REGISTRY_HTTP_ADDR=0.0.0.0:4000 -v registry:/var/lib/registry registry:2
```

<details close>
  <summary>Output</summary>

  ```sh
Unable to find image 'registry:2' locally
2: Pulling from library/registry
1cc3d825d8b2: Pull complete
85ab09421e5a: Pull complete
40960af72c1c: Pull complete
e7bb1dbb377e: Pull complete
a538cc9b1ae3: Pull complete
Digest: sha256:ac0192b549007e22998eb74e8d8488dcfe70f1489520c3b144a6047ac5efbe90
Status: Downloaded newer image for registry:2
7ebfa449f6542e04be94dfe414e971d72c179b4e45b2f3f00c25185c3006176b
```
</details>

3. Pull images from docker hub:
```sh
 $ cp -r /usr/local/share/kolla/etc_examples/kolla/ /etc/
 $ kolla-ansible -i /etc/kolla/all-in-one  pull
 ```

<details close>
  <summary>Output</summary>

  ```sh 
 
Pulling Docker images : ansible-playbook -e @/etc/kolla/globals.yml  -e @/etc/kolla/passwords.yml -e CONFIG_DIR=/etc/kolla  -e kolla_action=pull /usr/local/share/kolla-ansible/ansible/site.yml  --inventory /etc/kolla/all-in-one
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

PLAY [Gather facts for all hosts] ************************************************************************************************************************************************************************************************************
...
TASK [common : include_tasks] ****************************************************************************************************************************************************************************************************************
included: /usr/local/share/kolla-ansible/ansible/roles/common/tasks/pull.yml for localhost

TASK [service-images-pull : common | Pull images] ********************************************************************************************************************************************************************************************
...
TASK [service-images-pull : memcached | Pull images] *****************************************************************************************************************************************************************************************
changed: [localhost] => (item=memcached)
...
TASK [service-images-pull : keystone | Pull images] ******************************************************************************************************************************************************************************************
...
PLAY RECAP *********************************************************************
localhost                  : ok=33   changed=4    unreachable=0    failed=0    skipped=52   rescued=0    ignored=0
...
```
</details>


4. Push the Kolla images to the local Docker registry:

```sh
$ docker images | grep kolla | grep -v local | awk '{print $1,$2}' | while read -r image tag; do
    docker tag ${image}:${tag} localhost:4000/${image}:${tag}
    docker push localhost:4000/${image}:${tag}
done
```
<details close>
  <summary>Output</summary>

  ```sh 
The push refers to repository [localhost:4000/quay.io/openstack.kolla/kolla-toolbox]
54d649b3cff5: Pushed
27f380fa24ee: Pushed
3457e977a806: Pushed
b2ca7f36f982: Pushed
177e125c72fa: Pushed
a9d0e5fa10a0: Pushed
e2ad077ae545: Pushed
74d7c2b0f001: Pushed
86b9731b59fc: Pushed
54fc7df35234: Pushed
4a9f87f1bfc1: Pushed
...
9709badbf4c5: Pushed
f31e6842dfbe: Pushed
34ab490fb528: Pushed
ba310fe303b2: Pushed
1cb8d67c4dcb: Pushed
9684759379be: Pushed
b8341fb921aa: Pushed
fec7c6b4fbb0: Pushed
master-rocky-9: digest: sha256:cc27341fe80f1a6d6441fd1a81c05b3cd194431ccb04e3db5d6fd5d56e2963d4 size: 10512
The push refers to repository [localhost:4000/quay.io/openstack.kolla/mariadb-server]
0e883e1ea988: Pushed
b2af7db96489: Pushed
dac7d7a35df3: Pushed
3eb700f977f4: Pushed
c37d7231fdc8: Pushed
65f8253f10ec: Pushed
c976943928d1: Pushed
...
602353fdb8aa: Pushed
deba5123e2be: Pushed
1b3a4aa208c2: Pushed
edcc4217bbbd: Pushed
120ded57e1ea: Mounted from openstack.kolla/horizon
cfb18f4d0574: Mounted from openstack.kolla/horizon
48d2efbc43e3: Mounted from openstack.kolla/horizon
a03f6eff7b8e: Mounted from openstack.kolla/horizon
bea8b31e8aa7: Mounted from openstack.kolla/horizon
cc86fba08421: Mounted from openstack.kolla/horizon
...
master-rocky-9: digest: sha256:bab04a4d68825b0e04a55aa806b5b389bb2324934fab3749936bd28c2055aa82 size: 10714
The push refers to repository [localhost:4000/quay.io/openstack.kolla/haproxy]
7892e1906873: Pushed
287fcc86275d: Pushed
6dadf287fa67: Pushing [==================================================>]  12.54MB
73e30e55f6aa: Pushed
72637dbe1c66: Pushing [===============================================>   ]  14.93MB/15.55MB
4e9efe81ca16: Pushed
f0e0fd7e90a1: Mounted from openstack.kolla/nova-api
e08eef10480e: Mounted from openstack.kolla/nova-api
f1b0b4c5e194: Mounted from openstack.kolla/nova-api
9b2803c6b3a0: Mounted from openstack.kolla/nova-api
796753d46d4d: Mounted from openstack.kolla/nova-api
fe482168720f: Mounted from openstack.kolla/nova-api
2a61f2fdd58b: Mounted from openstack.kolla/nova-api
9f21580af66a: Mounted from openstack.kolla/nova-api
120ded57e1ea: Mounted from openstack.kolla/nova-api
...

...
```
</details>

5. Check if the images have been successfully pushed to our the created local registry:

```sh
$ curl -X GET http://localhost:4000/v2/_catalog | python3 -m json.tool
```

<details close>
  <summary>Output</summary>

  ```yaml
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   993  100   993    0     0  37167      0 --:--:-- --:--:-- --:--:-- 38192
{
    "repositories": [
        "openstack.kolla/cron",
        "openstack.kolla/fluentd",
        "openstack.kolla/glance-api",
        "openstack.kolla/haproxy",
        "openstack.kolla/heat-api",
        "openstack.kolla/heat-api-cfn",
        "openstack.kolla/heat-engine",
        "openstack.kolla/horizon",
        "openstack.kolla/keystone",
        "openstack.kolla/keystone-fernet",
        "openstack.kolla/keystone-ssh",
        "openstack.kolla/kolla-toolbox",
        "openstack.kolla/mariadb-server",
        "openstack.kolla/memcached",
        "openstack.kolla/neutron-dhcp-agent",
        "openstack.kolla/neutron-l3-agent",
        "openstack.kolla/neutron-metadata-agent",
        "openstack.kolla/neutron-openvswitch-agent",
        "openstack.kolla/neutron-server",
        "openstack.kolla/nova-api",
        "openstack.kolla/nova-compute",
        "openstack.kolla/nova-conductor",
        "openstack.kolla/nova-libvirt",
        "openstack.kolla/nova-novncproxy",
        "openstack.kolla/nova-scheduler",
        "openstack.kolla/nova-ssh",
        "openstack.kolla/openvswitch-db-server",
        "openstack.kolla/openvswitch-vswitchd",
        "openstack.kolla/placement-api",
        "openstack.kolla/proxysql",
        "openstack.kolla/rabbitmq"
    ]
}

```
</details>


6. Adjust the ```/etc/kolla/globals.yml``` file  the IP address and port on which the registry is listening:
```sh
$ sed -i 's/^#docker_registry:.*/docker_registry: 10.0.2.15:4000/g' /etc/kolla/globals.yml
$ sed -i 's/^#docker_registry_insecure:.*/docker_registry_insecure: yes/g' /etc/kolla/globals.yml
```

Now all OpenStack services containers will be pulled from the private Docker registry. 

7. Optionally, clean up the pulled Docker images from default registry used by kolla-ansible ```quay.io/openstack.kolla```.  This can be done by removing the repository/tag of  ```quay.io/openstack.kolla``` using images names:

```sh
$ docker rmi $(docker images  | grep -v localhost:4000  | grep quay.io/openstack.kolla | awk '{print $1":"$2}')
```

<details close>
  <summary>Output</summary>

  ```sh
Untagged: quay.io/openstack.kolla/keystone:master-rocky-9
Untagged: quay.io/openstack.kolla/keystone@sha256:10ceaee146dc6040407ae61edc0d27c8f535815069130b9ca42c4368ab3fb059
Untagged: quay.io/openstack.kolla/nova-compute:master-rocky-9
Untagged: quay.io/openstack.kolla/nova-compute@sha256:feb2378a12cc48c15753cf1364e944ee3912aa50ca94342984d4d1ba047ec618
Untagged: quay.io/openstack.kolla/neutron-server:master-rocky-9
Untagged: quay.io/openstack.kolla/neutron-server@sha256:5c840cb90affa73607727613fa0550ca7d52d0e3b81cde4f2e16a7de7f511bbb
Untagged: quay.io/openstack.kolla/neutron-dhcp-agent:master-rocky-9
Untagged: quay.io/openstack.kolla/neutron-dhcp-agent@sha256:e770ed73b5601c42425aae6d1cf6c79edd38171e1a137270895a38e4b1f7edfc
Untagged: quay.io/openstack.kolla/neutron-l3-agent:master-rocky-9
Untagged: quay.io/openstack.kolla/neutron-l3-agent@sha256:a23dc43eb20bc22842e59354705164824ab1d2e8a66a9d419a2e577339b3edf4
Untagged: quay.io/openstack.kolla/keystone-fernet:master-rocky-9
Untagged: quay.io/openstack.kolla/keystone-fernet@sha256:d29730d8fb1ac45cd944094e2602264f36f954528e007aa846e47f9daccae22f
Untagged: quay.io/openstack.kolla/neutron-openvswitch-agent:master-rocky-9
Untagged: quay.io/openstack.kolla/neutron-openvswitch-agent@sha256:3ca91ff09bdfbf000e3dbfe706a7e25c0783269aa89e1c5084e01320e87b45d3
Untagged: quay.io/openstack.kolla/neutron-metadata-agent:master-rocky-9
...
Untagged: quay.io/openstack.kolla/proxysql@sha256:d4726fad9fba2523a22e050fa59e80476cdf60f465db3d3bd5c8c51bb2f6ec90
Untagged: quay.io/openstack.kolla/haproxy:master-rocky-9
Untagged: quay.io/openstack.kolla/haproxy@sha256:9e1a3b199825d2f8a9879fdc8e95b6b4ef2074833584c4d0ab063c73efd9f922
Untagged: quay.io/openstack.kolla/cron:master-rocky-9
Untagged: quay.io/openstack.kolla/cron@sha256:2afcf15dffb82952dd5d44fcdf8169daac7c6f2ae1e2e5590a6fd9e959623cd8

```
</details>

Double check the docker images using local repository:

```sh
$ docker images
```

<details close>
  <summary>Output</summary>

  ```sh
REPOSITORY                                                 TAG              IMAGE ID       CREATED         SIZE
localhost:4000/openstack.kolla/keystone                    master-rocky-9   ff390bd35ef5   9 hours ago     1.28GB
localhost:4000/openstack.kolla/nova-compute                master-rocky-9   0fef86b10cbe   9 hours ago     2.03GB
localhost:4000/openstack.kolla/neutron-server              master-rocky-9   e2483db9b9eb   9 hours ago     1.41GB
localhost:4000/openstack.kolla/neutron-dhcp-agent          master-rocky-9   8589f4f2e6a5   9 hours ago     1.4GB
localhost:4000/openstack.kolla/neutron-l3-agent            master-rocky-9   480341095aaa   9 hours ago     1.43GB
localhost:4000/openstack.kolla/keystone-fernet             master-rocky-9   c0ab496f5dcd   9 hours ago     1.25GB
localhost:4000/openstack.kolla/neutron-openvswitch-agent   master-rocky-9   8940772dced2   9 hours ago     1.4GB
localhost:4000/openstack.kolla/neutron-metadata-agent      master-rocky-9   da4bf21b3940   9 hours ago     1.4GB
localhost:4000/openstack.kolla/keystone-ssh                master-rocky-9   44772d62a4f7   9 hours ago     1.25GB
localhost:4000/openstack.kolla/nova-ssh                    master-rocky-9   67107ff029e9   9 hours ago     1.48GB
localhost:4000/openstack.kolla/nova-novncproxy             master-rocky-9   eb44a4919c12   9 hours ago     1.57GB
localhost:4000/openstack.kolla/nova-scheduler              master-rocky-9   a415993b6a5e   9 hours ago     1.46GB
localhost:4000/openstack.kolla/glance-api                  master-rocky-9   1d6002abbcfe   9 hours ago     1.32GB
localhost:4000/openstack.kolla/nova-conductor              master-rocky-9   0acac0bd1576   9 hours ago     1.46GB
localhost:4000/openstack.kolla/nova-api                    master-rocky-9   4f5dd6b73404   9 hours ago     1.46GB
localhost:4000/openstack.kolla/heat-api                    master-rocky-9   748c3bdc90ea   9 hours ago     1.24GB
localhost:4000/openstack.kolla/placement-api               master-rocky-9   216e72f18ccd   9 hours ago     1.17GB
localhost:4000/openstack.kolla/heat-engine                 master-rocky-9   c16a56654f57   9 hours ago     1.24GB
localhost:4000/openstack.kolla/heat-api-cfn                master-rocky-9   46b49a823bcb   9 hours ago     1.24GB
localhost:4000/openstack.kolla/horizon                     master-rocky-9   03002f792742   9 hours ago     1.38GB
localhost:4000/openstack.kolla/kolla-toolbox               master-rocky-9   be9fcf4b0bb9   9 hours ago     1.16GB
localhost:4000/openstack.kolla/openvswitch-vswitchd        master-rocky-9   2cc8a65bcd43   9 hours ago     717MB
localhost:4000/openstack.kolla/openvswitch-db-server       master-rocky-9   40bde37a6b96   9 hours ago     717MB
localhost:4000/openstack.kolla/mariadb-server              master-rocky-9   09640f3ccb30   9 hours ago     1.09GB
localhost:4000/openstack.kolla/nova-libvirt                master-rocky-9   22e98cd196eb   9 hours ago     1.43GB
localhost:4000/openstack.kolla/memcached                   master-rocky-9   6e2945af43e2   9 hours ago     672MB
localhost:4000/openstack.kolla/fluentd                     master-rocky-9   20063b76626f   9 hours ago     887MB
localhost:4000/openstack.kolla/rabbitmq                    master-rocky-9   221d84cd11cf   9 hours ago     707MB
localhost:4000/openstack.kolla/proxysql                    master-rocky-9   1b1d1d1d9cc9   9 hours ago     772MB
localhost:4000/openstack.kolla/cron                        master-rocky-9   1a474460ab2d   9 hours ago     644MB
localhost:4000/openstack.kolla/haproxy                     master-rocky-9   50d99640da1f   9 hours ago     659MB
registry                                                   2                75ef5b734af4   12 months ago   25.4MB

```
</details>


## 3.Setting Up The CI/CD Pipeline:
1. Update the local repo system and install OpenJDK:
```sh
$ sudo apt-get update -y
$ sudo apt install openjdk-11-jdk -y
```
2. Import the GPG key to verify the package integrity:
```sh
$ curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
3. Add Jenkins package repository to the system source list with the authentication key:
```sh
$ echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```
4. Updatethe system repository and install Jenkins:
```sh
$ sudo apt-get update -y
$ sudo apt install jenkins -y
```
5. Check Jenkins daemon is running:
```sh
$ sudo systemctl status jenkins
```

<details close>
  <summary>Output</summary>

  ```sh
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-10-22 14:51:56 UTC; 2min 30s ago
   Main PID: 7536 (java)
      Tasks: 45 (limit: 9274)
     Memory: 540.7M
        CPU: 50.627s
     CGroup: /system.slice/jenkins.service
             └─7536 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080
             ....

```
</details>

6. Open a browser and point to the Jenkins host IP (http://your_jenkins_ip:8080)

![WelcomeJenkins](IMG/Jenkins01.png)



7. Grab the first Jenkins admin user from ```/var/lib/jenkins/secrets/initialAdminPassword``` and paste it in the ```Unlock Jenkins``` interface:

8. Click on Continue and choose ```Install suggested plugins ```

![WelcomeJenkins](IMG/Jenkins02.png)
![WelcomeJenkins](IMG/Jenkins03.png)

9. Create an Admin user and click on ```Save and Continue```:
--IMG--Jenkins04

10. Save the Jenkins URL (http://4.185.55.49:8080/) and click on ```Save and Finish```:
--IMG--Jenkins05

11. Click on  ```Start using Jenkins```
![WelcomeJenkins](IMG/Jenkins06.png)
![WelcomeJenkins](IMG/Jenkins07.png)

12. Click  on ```Manage Jenkins``` to start adding Jenkins plugins:
![WelcomeJenkins](IMG/Jenkins08.png)

12. From the ```Manage Jenkins``` page, click on ```Plugins```. From the Plugins list on the left side, click on ```Available plugins```:
![WelcomeJenkins](IMG/Jenkins09.png)

13. In the search bar, search for ```github``` and select the plugin names as listed below:
![WelcomeJenkins](IMG/Jenkins10.png)

> [!NOTE]
> Some newer versions of Jenkins come with Github plugins already installed that can be found in ```Installed plugins``` section.

14. Configure credential key for Jenkins user to access the Github repository. Navigate to the Jenkins main dashboard, and click on ```Credentials```:
![WelcomeJenkins](IMG/Jenkins11.png)

15. Cick on ```System```under ```Stores scoped to Jenkins```:
![WelcomeJenkins](IMG/Jenkins12.png)

16. Click next on ```Global crendentials (unrestricted)```:
![WelcomeJenkins](IMG/Jenkins13.png)

17. To add the Private Key in the ```New credentials``` page, grab the SSH key from the Jenkins server. If no SSH keys exist, generate new ones using command lines in the Jenkins server:

```sh
$ su - jenkins
$ ssh-keygen -o -t rsa -C
```
<details close>
  <summary>Output</summary>

  ```sh
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa):
Created directory '/var/lib/jenkins/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa
Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:y1/LgP9uaSPBlXdtdTd83E1uvCt9RU+sv/4qRBDUBjU jenkins@ubuntu2204.localdomain
The key's randomart image is:
+---[RSA 3072]----+
|         .+=E .o+|
|          . o. *O|
|           o .  #|
|            + .=*|
|        S. o ..+o|
|       . oo . ..o|
|        + .o.o oo|
|         o.+*.. o|
|          o*=oo+o|
+----[SHA256]-----+

```
</details>

The public key and private keys are generated by default under  ```~/.ssh/```


18. Copy the content of the private key ```~/.ssh/id_rsa``` in the ```New crendials```, ```Private Key``` section. Make sure to select ```SSH Username with private key```from the ```kind```drop down list:
![WelcomeJenkins](IMG/Jenkins14.png)

19. Click on ```Create```and new credentials should be listed in the ```Global Credentials (unrestricted)``` tab:
 ![WelcomeJenkins](IMG/Jenkins15.png)

20. Add the SSH public key to your own Github repository. Login to your Github repository and in the ```Account settings```, select ```SSH and GPG keys```:
 ![WelcomeJenkins](IMG/Jenkins16.png)

21. Click on ```New SSH Key``` and paste the Public Key content from the Jenkins server:
 ![WelcomeJenkins](IMG/Jenkins17.png)

22. Click on ```Add SSH Key``` and it should be listed on the SSH Keys in the Github account keys list.  Navigate to the Jenkins main dashboard, and click on ```Create a job```:
![WelcomeJenkins](IMG/Jenkins18.png)

15. Enter a name of the new Job and choose ```Pipeline```:
![WelcomeJenkins](IMG/Jenkins19.png)

16. Click on ```Ok```and add a description of the Pipeline configuration:
![WelcomeJenkins](IMG/Jenkins20.png)

17. Scroll down to the ```Pipeline```section. In the ```Definition``` drop down list, select ```Pipeline Script from SCM```
![WelcomeJenkins](IMG/Jenkins21.png)

18. Enter the the URL of your Github repository and and select the created Credentials from the drop down list:
![WelcomeJenkins](IMG/Jenkins22.png)

19. Enter ```Save``` and click on ```Build Now``` in the left tab:

![WelcomeJenkins](IMG/Jenkins23.png)

20. Jenkins should start the pipeline and monitor the pipeline stages form the ```Console Output```:
![WelcomeJenkins](IMG/Jenkins24.png)
![WelcomeJenkins](IMG/Jenkins25.png)


## 4. Integrating Security:
### 4.1. Install Jenkins Plugin:

1. Navigate to ```Dashboard```--> ```Manage Jenkins``` --> ```Plugins```and select ```Available Plugins```. Type in the seach bar ```Anchore``` and select ```Anchore Container Image Scanner``` then click on ```Install```:
![WelcomeAnchore](IMG/Anchore01.png)
![WelcomeAnchore](IMG/Anchore02.png)

### 4.2. Configure Anchore: 

1. Download Anchore CLI:
```sh
$ git clone https://github.com/anchore/anchore-cli
```

<details close>
  <summary>Output</summary>

  ```sh
Cloning into 'anchore-cli'...
remote: Enumerating objects: 2541, done.
remote: Counting objects: 100% (280/280), done.
remote: Compressing objects: 100% (143/143), done.
remote: Total 2541 (delta 124), reused 263 (delta 119), pack-reused 2261 (from 1)
Receiving objects: 100% (2541/2541), 657.59 KiB | 3.96 MiB/s, done.
Resolving deltas: 100% (1417/1417), done.
```
</details>

2. Install Anchore CLI:

```sh
$ cd anchore-cli
$ pip install --user --upgrade .
```

<details close>
  <summary>Output</summary>

  ```sh
Processing /root/anchore-cli
  Preparing metadata (setup.py) ... done
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x7f4a08218880>: Failed to establish a new connection: [Errno -2] Name or service not known')': /simple/click/
Collecting Click==8.0.1
  Downloading click-8.0.1-py3-none-any.whl (97 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.4/97.4 KB 758.5 kB/s eta 0:00:00
Requirement already satisfied: PyYAML==5.4.1 in /usr/lib/python3/dist-packages (from anchorecli==0.9.4) (5.4.1)
...
...
Successfully installed Click-8.0.1 anchorecli-0.9.4 charset-normalizer-2.0.12 prettytable-2.2.0 python-dateutil-2.8.2 requests-2.26.0 urllib3-1.26.6 wcwidth-0.2.13
```
</details>



3. Download the Docker compose of the Anchore image or copy/paste content of the file [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/docker-compose.yaml):
```sh
$ curl https://engine.anchore.io/docs/quickstart/docker-compose.yaml > docker-compose.yaml
```
<details close>
  <summary>Output</summary>

  ```sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4331    0  4331    0     0  18684      0 --:--:-- --:--:-- --:--:-- 18748
```
</details>



4. Install ```docker-compose``` if not installed yet:

```sh
$ sudo apt install docker-compose
```

 5. Deploy the docker container from the `docker-compose.yaml` file to run the Anchore engine:
 ```sh
 $ docker-compose up -d
 ```

<details close>
  <summary>Output</summary>

  ```sh
Creating network "root_default" with the default driver
Creating volume "root_anchore-db-volume" with default driver
Pulling db (postgres:9)...
9: Pulling from library/postgres
1cb79db8a9e7: Pull complete
f6bae7873dd7: Pull complete
8f7722dc50a7: Pull complete
e8622b8cb6f3: Pull complete
d6d74bba3a57: Extracting [==================================================>]  6.186MB/6.186MB
874d4d2a09fd: Download complete
2d87c3a4038c: Download complete
f955a6cf127b: Download complete
f62dc55c568d: Download complete
4e2c4902efbd: Download complete
01c676df543a: Download complete
1e3d335ef0b7: Download complete
11087f2b0d87: Download complete
4b9a74ac6ea0: Download complete
Digest: sha256:caddd35b05cdd56c614ab1f674e63be778e0abdf54e71a7507ff3e28d4902698
Status: Downloaded newer image for postgres:9
Pulling catalog (anchore/anchore-engine:v1.0.0)...
v1.0.0: Pulling from anchore/anchore-engine
262268b65bd5: Downloading [=========================>                         ]  43.02MB/83.37MB
06038631a24a: Download complete
65c7dbde5391: Download complete
7bc4d5530020: Downloading [==========================================>        ]  39.32MB/45.82MB
1598dedc85bc: Waiting
3f9a12d5f96a: Waiting
f15685c5ee70: Waiting
d84a851b2101: Waiting
...
Digest: sha256:68e09d5ff695b46f24de3d7610f88dfc14b77b38758bbd68d03350dbae3deb0f
Status: Downloaded newer image for anchore/anchore-engine:v1.0.0
Creating root_db_1 ... done
Creating root_catalog_1 ... done
Creating root_analyzer_1      ... done
Creating root_api_1           ... done
Creating root_queue_1         ... done
Creating root_policy-engine_1 ... done
```
</details>

Note the creation of different Anchore services running in Docker containers:

```sh
$ docker ps -a
```

<details close>
  <summary>Output</summary>

  ```sh
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                    PORTS                    NAMES
b685fabb7cf8   anchore/anchore-engine:v1.0.0   "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes (healthy)   0.0.0.0:8228->8228/tcp   root_api_1
d8954f433db1   anchore/anchore-engine:v1.0.0   "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes (healthy)   8228/tcp                 root_queue_1
ae4647a449a7   anchore/anchore-engine:v1.0.0   "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes (healthy)   8228/tcp                 root_analyzer_1
639bae3a6447   anchore/anchore-engine:v1.0.0   "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes (healthy)   8228/tcp                 root_policy-engine_1
4786257fdafb   anchore/anchore-engine:v1.0.0   "/docker-entrypoint.…"   12 minutes ago   Up 12 minutes (healthy)   8228/tcp                 root_catalog_1
f126374d667e   postgres:9                      "docker-entrypoint.s…"   13 minutes ago   Up 12 minutes (healthy)   5432/tcp                 root_db_1
```
</details>


### 4.3. Create The Pipeline: 

1. Login to the Jenkins dashboard and navigate to ```Manage Jenkins```--> ```System```. Scroll down to the ```Anchore Container Image Scanner```section and add the ```Engine URL```, ```Engine Username``` and ```Engine Password``` values that can be found in ```docker-compose.yaml```file:

![WelcomeJenkins](IMG/Anchore03.png)

2. Navigate to the main Jenkins dashboard, and click on ```Create a new Job```:
![WelcomeJenkins](IMG/Anchore04.png)

3. Choose between a ```Freestyle project``` or ```Pipeline```. The ```Freestyle project``` includes ```Build Steps```that can be configured for ```Anchore Container Scanner``` pipeline stages:
![WelcomeJenkins](IMG/Anchore05.png)

The following steps will use a ```Pipeline``` job:

![WelcomeJenkins](IMG/Anchore06.png)

Click  `OK`

4. Create or copy the pipeline script that can be found [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter02/JenkinsfileAnchore):
![WelcomeJenkins](IMG/Anchore07.png)

5. Click on ```Save```and the pipeline should start building images, Anchore scans and pushes images if the vulnerability checks pass otherwise it fails. 


# 5. Troubleshooting:
## Deploy or Reconfigure Failure: 
### Error: Not supported URL scheme http+docker

```sh
...
 File \"/usr/local/lib/python3.10/dist-packages/docker/api/client.py\", line 221, in _retrieve_server_version\\n    raise DockerException(\\ndocker.errors.DockerException: Error while fetching server API version: Not supported URL scheme http+docker\\n'"}
 ...
```
Change the Docker SDK package version for ```requests``` in ```/.ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml```file from ```requests<2.32``` to ```requests==2.31```:

```sh
$ vim .ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml
```

```yaml
docker_sdk_pip_packages:
  - "docker>=6.0.0,<7.0.0"
#  - "requests<2.32"
  - "requests==2.31.0"
  - "dbus-python"
```
Install the updated version:

```sh
$ pip3 install requests==2.31.0 
```

Rerun the deployment pipeline.

### Error: Cron, kolla-box and fluentd containers fails to run during deployment

```sh
...
Failed: [localhost] (item={'key': 'fluentd', 'value': {'container_name': 'fluentd', 'group': 'fluentd', 'enabled': True, 'image': 'quay.io/openstack.kolla/fluentd:master-rocky-9', 'environment': {'KOLLA_CONFIG_STRATEGY': 'COPY_ALWAYS'}, '
...
...
failed: [localhost] (item={'key': 'kolla-toolbox', 'value': {'container_name': 'kolla_toolbox', 'group': 'kolla-toolbox', 'enabled': True, 'image': 'quay.io/openstack.kolla/kolla-toolbox:master-rocky-9', 'environment': {'ANSIBLE_NOCOLOR': '1',
...
...
failed: [localhost] (item={'key': 'cron', 'value': {'container_name': 'cron', 'group': 'cron', 'enabled': True, 'image': 'quay.io/openstack.kolla/cron:master-rocky-9', 'environment': {'KOLLA_LOGROTATE_SCHEDULE': 'daily'}, 'volumes': [
...
```

Change the Docker SDK package version for ```requests``` in ```/.ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml```file from ```requests<2.32``` to ```requests==2.31```:

```sh
# vim /.ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml
```

```sh
docker_sdk_pip_packages:
  - "docker>=6.0.0,<7.0.0"
#  - "requests<2.32"
  - "requests==2.31.0"
  - "dbus-python"
```
Install the updated version:

```sh
$ pip3 install requests==2.31.0 
```

Rerun the deployment pipeline.

### Error: Precheck stage TASK [common : Check common containers]

```sh
failed: [localhost] (item={'key': 'fluentd', 'value': {'container_name': 'fluentd', 'group': 'fluentd', 'enabled': True, 'image': '10.0.2.15:4000/openstack.kolla/fluentd:master-rocky-9', 'environment': {'KOLLA_CONFIG_STRATEGY': 'COPY_ALWAYS'}, 'volumes': ['/etc/kolla/fluentd/:/var/lib/kolla/config_files/:ro', '/etc/localtime:/etc/localtime:ro', '/etc/timezone:/etc/timezone:ro', 'kolla_logs:/var/log/kolla/', 'fluentd_data:/
...
self._version = self._retrieve_server_version()\\n  File \"/usr/lib/python3/dist-packages/docker/api/client.py\", line 221, in _retrieve_server_version\\n    raise DockerException(\\ndocker.errors.DockerException: Error while fetching server API version: (\\'Connection aborted.\\', ConnectionRefusedError(111, \\'Connection refused\\'))\\n'"
...

```

Check if the Docker daemon is running:

```sh
$ systemctl status docker
```
<details close>
  <summary>Output</summary>

  ```sh
× docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Wed 2024-10-23 11:48:16 UTC; 1min 47s ago
TriggeredBy: × docker.socket
       Docs: https://docs.docker.com
    Process: 27805 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
   Main PID: 27805 (code=exited, status=1/FAILURE)
        CPU: 115ms

Oct 23 11:48:16 ubuntu2204.localdomain systemd[1]: docker.service: Scheduled restart job, restart counter is at 3.
Oct 23 11:48:16 ubuntu2204.localdomain systemd[1]: Stopped Docker Application Container Engine.
Oct 23 11:48:16 ubuntu2204.localdomain systemd[1]: docker.service: Start request repeated too quickly.
Oct 23 11:48:16 ubuntu2204.localdomain systemd[1]: docker.service: Failed with result 'exit-code'.
Oct 23 11:48:16 ubuntu2204.localdomain systemd[1]: Failed to start Docker Application Container Engine.

```
</details>

Start the Docker daemon and double check the status:
```sh
$ systemctl start docker
$ systemctl status docker
```
<details close>
  <summary>Output</summary>

  ```sh
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-10-23 11:54:01 UTC; 2s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 29335 (dockerd)
      Tasks: 11
     Memory: 27.0M
        CPU: 3.387s
     CGroup: /system.slice/docker.service
             └─29335 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Oct 23 11:53:56 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:53:56.761275747Z" level=info msg="detected 127.0.0.53 nameserver, assuming systemd-resolved, so using resolv.conf: /run/systemd/resolve/resolv.conf"
Oct 23 11:53:57 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:53:57.115828707Z" level=info msg="[graphdriver] using prior storage driver: overlay2"
Oct 23 11:53:58 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:53:58.308370041Z" level=info msg="Loading containers: start."
Oct 23 11:53:59 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:53:59.445052351Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP add>
Oct 23 11:53:59 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:53:59.776676248Z" level=warning msg="error locating sandbox id b8919b08bb2eaf578a22334615305e0c0b70dad4b99960bbe26aef26b1402af1: sandbox b8919b08bb2eaf578a2233461>
Oct 23 11:54:00 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:54:00.730711208Z" level=info msg="Loading containers: done."
Oct 23 11:54:01 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:54:01.345831519Z" level=info msg="Docker daemon" commit=41ca978 containerd-snapshotter=false storage-driver=overlay2 version=27.3.1
Oct 23 11:54:01 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:54:01.346799915Z" level=info msg="Daemon has completed initialization"
Oct 23 11:54:01 ubuntu2204.localdomain dockerd[29335]: time="2024-10-23T11:54:01.585754499Z" level=info msg="API listen on /run/docker.sock"
Oct 23 11:54:01 ubuntu2204.localdomain systemd[1]: Started Docker Application Container Engine.

```
</details>



### Error: kolla-ansible deploy hanging to start MariaDB service  
```sh
RUNNING HANDLER [mariadb : Wait for first MariaDB service port liveness] *******
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (10 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (9 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (8 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (7 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (6 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (5 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (4 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (3 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (2 retries left).
FAILED - RETRYING: [localhost]: Wait for first MariaDB service port liveness (1 retries left).
fatal: [localhost]: FAILED! => {"attempts": 10, "changed": false, "elapsed": 61, "msg": "Timeout when waiting for search string MariaDB in 10.0.2.15:3306"}

PLAY RECAP *********************************************************************
localhost                  : ok=64   changed=31   unreachable=0    failed=1    skipped=66   rescued=0    ignored=1
```

Check the Mariadb container logs: 

```sh
tail -f /var/log/kolla/mariadb/mariadb.log
```
<details close>
  <summary>Output</summary>

  ```sh
2024-10-23 13:53:02 0 [Note] Server socket created on IP: '10.0.2.15'.
2024-10-23 13:53:02 0 [ERROR] Can't start server: Bind on TCP/IP port. Got error: 98: Address already in use
2024-10-23 13:53:02 0 [ERROR] Do you already have another server running on port: 3306 ?
2024-10-23 13:53:02 0 [ERROR] Aborting'
```
</details>

`3306` seems it is being used by other process that can a from previous failed deployments. List the processes using the port: 


```sh
$ netstat -anp | grep 3306
```

<details close>
  <summary>Output</summary>

  ```sh
tcp        0      0 10.0.2.15:3306          10.0.2.15:38828         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:57404         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:50990         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:50994         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:38836         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:57388         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:38820         TIME_WAIT   -
tcp        0      0 10.0.2.15:3306          10.0.2.15:50978         TIME_WAIT   -
```
</details>

Kill the  processes using the port:

```sh
$ fuser 3306/tcp
```

Rerun the pipeline to deploy again the Mariadb containers. 


## Jenkins Pipeline Run
### Error: Jenkins User Permissions

```sh
ERROR:kolla.common.utils:Unable to connect to container engine daemon, exiting
```
Similar issue with permissions for Jenkins user can be fixed by upgrading the ```jenkins```user in the Deployer host by adjusting the ```sudoers``` file:

As root, edit the ```/etc/sudoers``` and add the following:
```sh
$ vim /etc/sudoers
```
```sh
...
 jenkins ALL=(ALL) NOPASSWD: ALL
... 
```

### Error: Jenkins Workspace source not found

```sh
/var/lib/jenkins/workspace/openstack-dev@tmp/durable-4228d9ca/script.sh.copy: 7: source: not found
```
Multiple runs can result on some stalling caches for each job execution  


Removing the temporary workspace and rerunning the job pipeline should solve the issue.


### Error:  Pipeline Jenkins failure (Step Bootstrap Server Script)


```sh
ERROR! the role 'openstack.kolla.baremetal' was not found in /usr/local/share/kolla-ansible/ansible/roles:/var/lib/jenkins/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/usr/local/share/kolla-ansible/ansible

The error appears to be in '/usr/local/share/kolla-ansible/ansible/kolla-host.yml': line 13, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  roles:
    - { role: openstack.kolla.baremetal,
      ^ here
```
The issue might appear during the pipeline stage ```Pipeline Jenkins failure (Step Bootstrap Server Script)```.
When installing the required packages, make sure to source the python virtual enviornment if being used and run:

```sh
$ kolla-ansible install-deps
```

## Anchore Engine VS Anchore Enterprise
The book uses ```Anchore Engine``` as a container inspection and analysis service. ```Anchore Engine``` has been moved to an ```Anchore Entreprise ``` edition and no longer maintained as an open-source project (https://github.com/anchore/anchore-engine).
At the time of publishing the book, the new Jenkins Anchore plugin is developed to use only ``` Anchore Entreprise``` as shown here (The chapter was released before the Plugin changed):

![WelcomeJenkins](IMG/TS1.png)

The latest updates of the Anchore Jenkins plugin can be found [here](https://plugins.jenkins.io/anchore-container-scanner/)

You can still use trial verison of ``` Anchore Entreprise``` for free as described [here](https://docs.anchore.com/3.0/docs/quickstart/)
It will be required to have:
- DockerHub account 
- License obtained from the Anchore account creation (trial license)

The ```Anchore Engine``` still can be used using `Anchore CLI` as the Jenkins plugins supports only ``` Anchore Entreprise``` since August 2024. 


## Anchore CLI
1. At the time of writing the book, the ```Anchore CLI``` open-source project is no longer maintained (10th July 2024) but can still be used either with  ```Anchore Engine``` or ```Anchore Entreprise ``` (https://github.com/anchore/anchore-cli).

To install the ```Anchore CLI``` client on the deployer host, make sure to run the following command line (for Ubuntu OS):

```sh
$ apt-get update
$ apt-get install python-pip
$ pip install anchorecli
$ export PATH="$HOME/.local/bin/:$PATH"
```
Check if ```Anchore CLI``` is correctly installed:

```sh
$ anchore-cli -h
```
<details close>
  <summary>Output</summary>

  ```sh
Usage: anchore-cli [OPTIONS] COMMAND [ARGS]...

Options:
  --config TEXT       Set the location of the anchore-cli yaml configuration
                      file
  --debug             Debug output to stderr
  --u TEXT            Username (or use environment variable ANCHORE_CLI_USER)
  --p TEXT            Password (or use environment variable ANCHORE_CLI_PASS)
  --url TEXT          Service URL (or use environment variable
                      ANCHORE_CLI_URL)
  --hub-url TEXT      Anchore Hub URL (or use environment variable
                      ANCHORE_CLI_HUB_URL)
  --api-version TEXT  Explicitly specify the API version to skip checking.
                      Useful when swagger endpoint is inaccessible
  --insecure          Skip SSL cert checks (or use environment variable
                      ANCHORE_CLI_SSL_VERIFY=<y/n>)
  --json              Output raw API JSON
  --as-account TEXT   Set account context for the command to another account
                      than the one the user belongs to. Subject to authz
  --version           Show the version and exit.
  -h, --help          Show this message and exit.

Commands:
  account           Account operations
  analysis-archive  Archive operations
  enterprise        Enterprise Anchore operations
  evaluate          Policy evaluation operations
  event             Event operations
  help
  image             Image operations
  policy            Policy operations
  query             Query operations
  registry          Registry operations
  repo              Repository operations
  subscription      Subscription operations
  system            System operations
```
</details>


2. ```Anchore CLI``` will reach by default the ```Anchore Engine``` or ```Anchore Entreprise ``` server at ```http://localhost/v1 ```  without a username and password for authentication. That can be set as envionment variables in Jenkins Deployer host:
```sh

$ export ANCHORE_CLI_URL=http://localhost:8228/v1

$ export ANCHORE_CLI_USER=admin

$ export ANCHORE_CLI_PASS=foobar
```

> [!CAUTION]
> Note that the pair ```admin/foobar``` are the default ```Anchore``` authentication variables and must be changed as a security best practice. 


3. Add the local Docker Registry using Anchore CLI:
```sh
anchore-cli registry add localhost:4000
```
> [!IMPORTANT]
> Make sure to change the registry host address that fits your configuration.


