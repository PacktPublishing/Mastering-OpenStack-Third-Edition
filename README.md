# Mastering-OpenStack
Mastering OpenStack Third Edition, Published by Packt

## About the Book

An in-depth paragraph about your project and overview of use.


## How to Use the Code

### Code Navigation

*Describe how the book code repository is organised*

> [!IMPORTANT]
> OpenStack Kolla-Ansible  code is still maintained project by OpenStack community and actively being developed. Upon a new OpenStack release, features or services can be introduced as well as deprecated and removed. For the best usage of the code in this book, it is highly recommended to have high level understanding of the Kolla-Ansible code structure that is described in the following sections.  


### Kolla-Ansible Code Structure 

*Describe generic layout of the code community with highlevel code structure - desired with a lucidchart diagram*

### OpenStack Deployment Approach 

*Describe which files will be used are used across all chapters, modification, a push to CI/CD pipeline for testing and then production*

> [!IMPORTANT]
> The deployment in any environment will depend on existing resources.

*Each chapter introduces different changes that can be deployed separately*

*Starting the OpenStack deployment with test environment that  run all services in one single host and does not require a CI/CD integration*

*Production environment would require multi nodes setup. Code changes must be maintained in central repository. A CI/CD pipeline must be built to faciliate OpenStack deployement, testing and capturing faster changes in separate environment *

*A diagram for generic OpenStack code pipeline and deployment in Production (multi node with multi environment)*

> [!IMPORTANT]
> Deployment in production environment would require more resources to minimum setup of core OpenStack services including compure, storage, network and memory hardware resources.

## Prerequisites

### Running Test/Local Environment:


### Running Pre-Production/Production Environment(s):

* Describe any prerequisites, libraries, OS version, etc., needed before installing program.
* ex. Windows 10

### Installing

* How/where to download your program
* Any modifications needed to be made to files/folders

### Executing program

* How to run the program
* Step-by-step bullets
```
code blocks for commands
```

## Help

Any advise for common problems or issues.
```
command to run if program contains helper info
```

## Authors

Contributors names and contact info



## Resources

* Kolla-Ansbile community project: 
    * GitHub [repostority](https://github.com/openstack/kolla-ansible).
    * Documentation [Latest](https://docs.openstack.org/kolla-ansible/latest/).

* OpenStack Releases Notes:
    * Services [Projects List and Versions](https://releases.openstack.org/dalmatian/index.html).