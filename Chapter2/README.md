# Chapter 2
Kicking Off the OpenStack Setup – The Right Way (DevSecOps)

## Description

The Chapter initiates the deployment of an OpenStack environment in different steps:
1. Test/Local environment: Running all components discussed in first chapter in a single machine for testing using Vagrant and VirtualBox.
2. Setup a deployer host running a CI/CD Tool using Jenkins 
3. Create CI/CD pipeline for the OpenStack code deployment. 
4. Include a security stage check in the CI/CD pipeline to scan and test OpenStack services running in Kolla containers before proceeding the deployment in the target environment
5. **Bonus Section 1**: Guide through the different requirements to create a first instance.
6. **Bonus Section 2**: Troubleshooting of most encoutred issues when running an OpenStack deployment.  
5. **Bonus Section 3**: 

## 1.Deployment in a Local Environment:
### System and Software Requirements:

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

### Code - How-To:

1. Update the package lists from the repositories of Ubuntu OS:
```
$ sudo apt-get update
```

2. Install ```Virtualbox``` in the local machine:
```
$ sudo apt install -y virtualbox virtualbox-ext-pack
```

3. Install the Vagrant version ```2.4.1```:
```
$ wget https://releases.hashicorp.com/vagrant/2.4.1/vagrant-2.4.1-1.x86_64.rpm
$ sudo apt install rpm
$ sudo rpm -i vagrant-2.4.1-1.x86_64.rpm
```

4. Check the installed version of ```vagrant```:
```
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

```
$ vagrant plugin install vagrant-disksize
```

6. Create optionally a shared directory for Vagrant guest hosts:

```
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
```
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

```
$ vagrant ssh
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

10. Once logged in, install the following packages and dependencies:

```
$ sudo -i
# apt update -y
# apt install python3-dev libffi-dev gcc libssl-dev python3-venv
# apt install python3-pip
```

11. Create optionally a Python virtual envionment:

```
# python3 -m venv local
# source local/bin/activate
```

12. Install ```pip```:

```
(local)# pip install -U pip
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

13. Install Ansible core:
```
(local)# pip install 'ansible-core>=2.16,<2.17.99'
```

14. Install kolla-ansible:
```
(local)# pip install git+https://opendev.org/openstack/kolla-ansible@master
(local)# kolla-ansible install-deps
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

15. Create a new directory locally to prepare for the kolla-ansible run:
```
(local)# mkdir -p /etc/kolla
(local)# chown $USER:$USER /etc/kolla
```

16. Step ```14``` should have downloaded example files of kolla-ansibe repository that should be located under ```/local/share/kolla-ansible/```. Copy skeleton files for both `all-in-one` inventory and ```globals.yaml``` files under ```/etc/kolla```directory:

```
(local)# cp -r local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla 
(local)# cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla 
```

17. Create and copy the content of ```/etc/kolla/globals.yaml``` file provided [here](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/blob/main/Chapter2/etc/kolla/globals.yml).
You can also run the following command lines to get the same settings in the ```/etc/kolla/globals.yaml``` file  :
<br />

```
sed -i 's/^#kolla_base_distro:.ls*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/kolla/globals.yml
sed -i 's/^#network_interface:.*/network_interface: "eth0"/g' /etc/kolla/globals.yml
sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "eth1"/g' /etc/kolla/globals.yml
sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "10.0.2.15"/g' /etc/kolla/globals.yml
```

18. Generate the OpenStack services secretes:

```
# kolla-genpwd -p /etc/kolla/passwords.yml
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>


19. Bootstrap the OpenStack services in ```all-in-one``` configuration:
```
# kolla-ansible -i /etc/kolla/all-in-one bootstrap-servers
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

20. Run The Pre-checks script in ```all-in-one``` configuration:
```
# kolla-ansible -i /etc/kolla/all-in-one prechecks
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

21. Run the OpenStack services deployment in ```all-in-one``` configuration:
```
# kolla-ansible -i /etc/kolla/all-in-one deploy
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

22. Verify that all containers are up and running and in ```healthy``` status:

```
# docker ps -a
```

![List Container Services](IMG/docker-ps-a-all.png)


23. Run post deploy script to generate cloud configuration file:
```
# kolla-ansible -i /etc/kolla/all-in-one post-deploy
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

22. Check the config file generated under ```/etc/kolla/``` named ```clouds.yaml```:

```
# cat /etc/kolla/clouds.yaml
```
<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

23. You can also source the generated file ```admin_openrc.sh``` to start interacting with OpenStack APIs:
```
# source /etc/kolla/admin_openrc.sh
```
24. To interact with the dashboard for ```admin``` user, capture the password from either the  ```clouds.yaml```,  ```admin_openrc.sh``` or simply from ```passwords.yml``` file:

```
# grep keystone_admin_password /etc/kolla/passwords.yml
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

25. Open a browser on local host and navigate to the VIP address configured in the ```globals.yml``` file:

![List Container Services](IMG/WelcomeOS.png)
![List Container Services](IMG/Main-Dashboard-UI2.png)


26. To interact with OpenStack services using CLI, install the OpenStack client CLI tools:

```
# pip install python-openstackclient
```

27. Make sure the variables in the ```admin-openrc.sh``` are sourced and verify the OpenStack CLI:
```
# openstack services list
```

<details close>
  <summary>Output</summary>

  ```sh
TBD
```
</details>

## 2. Prepare The Deployment Environment:
### 2.1. Create Own Repository:
### 2.2. Create Own Private Container Registry:

## 3.Setting CI/CD Pipeline:
### 3.1. Install Jenkins
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



### 3.2. Install Jenkins Plugins:
### 3.3. Create The Pipeline:


## 4. Integrating Security:
1. Install Jenkins Plugin:
2. Create The Pipeline: 


# 5. Troubleshooting:

### VirtualBox installation
### Vagrant installation

### Vagrant disksize plugin: 
Issue with Vagrant version below 2.X.X

### Vagrant UP Run 

### Vagrant SSH too long

### Vagrant new configuation breaks

### Openstack Deplyoment: Docker missing
sudo apt-get install python3-dev libffi-dev gcc libssl-dev docker -y  
OR apt install docker.io

## Deploy or reconfigure fails: 
### Error: Not supported URL scheme http+docker

```
...
 File \"/usr/local/lib/python3.10/dist-packages/docker/api/client.py\", line 221, in _retrieve_server_version\\n    raise DockerException(\\ndocker.errors.DockerException: Error while fetching server API version: Not supported URL scheme http+docker\\n'"}
 ...
```
Change the Docker SDK package version for ```requests``` in ```/.ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml```file from ```requests<2.32``` to ```requests==2.31```:

```
# vim /.ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml
```

```
docker_sdk_pip_packages:
  - "docker>=6.0.0,<7.0.0"
#  - "requests<2.32"
  - "requests==2.31.0"
  - "dbus-python"
```
And run the deployment again. 

### Error: Cron, kolla-box and fluentd containers fails to run during deployment

```
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

```
# vim /.ansible/collections/ansible_collections/openstack/kolla/roles/docker_sdk/defaults/main.yml
```

```
docker_sdk_pip_packages:
  - "docker>=6.0.0,<7.0.0"
#  - "requests<2.32"
  - "requests==2.31.0"
  - "dbus-python"
```
And run the deployment again. 

### Jenkins
1. Git authentification access 
jenkins user upgrade to sudoers users with NOPASSW: jenkins ALL=(ALL) NOPASSWD: ALL
Add /bin/bash for the Jenkins shell configuration to execute 

2. Pipeline failure: 
/var/lib/jenkins/workspace/openstack-dev@tmp/durable-4228d9ca/script.sh.copy: 7: source: not found

--> ReRun the Pipeline due to cache 

3. Pipeline Jenkins failure (Step Bootstrap Server Script)
ERROR! the role 'openstack.kolla.baremetal' was not found in /usr/local/share/kolla-ansible/ansible/roles:/var/lib/jenkins/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/usr/local/share/kolla-ansible/ansible

The error appears to be in '/usr/local/share/kolla-ansible/ansible/kolla-host.yml': line 13, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  roles:
    - { role: openstack.kolla.baremetal,
      ^ here



### Anchore
https://github.com/anchore/anchore-cli
Entreprise Edition Workaround


## Nova user Guide: Create VM

1. Prepare image using Cirros image (https://docs.openstack.org/mitaka/install-guide-obs/glance-verify.html) (https://docs.openstack.org/image-guide/obtain-images.html#cirros-test)
 the login account is cirros. The password is gocubsgo.

$ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img

2. Upload the image to Glance:

```
$ openstack image create 'Cirros' --file cirros-0.6.2-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```


<details close>
  <summary>Output</summary>

  ```sh

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
  ```

</details>

3. Veriy the image:

```
$ openstack image list
```
<details close>
  <summary>Output</summary>

  ```sh
+--------------------------------------+----------------------+--------+
| ID                                   | Name                 | Status |
+--------------------------------------+----------------------+--------+
| a0198d59-6b58-4885-8aa8-cf41dba0a898 | Cirros               | active |
| 9be4165a-45a7-4d55-b2ef-65ed8d243adc | manila-service-image | active |
+--------------------------------------+----------------------+--------+
```
</details>

