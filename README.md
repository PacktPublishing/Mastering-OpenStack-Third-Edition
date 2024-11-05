# Mastering OpenStack
## Implement the latest techniques for designing and deploying operational production-ready private clouds

## About the Book

An in-depth paragraph about your project and overview of use.


## How to Use the Code

### Code Navigation

The book uses mainly the kolla-ansible community [repostority](https://github.com/openstack/kolla-ansible).

You can check the branch naming standard used by the OpenStack community in the Github page by clicking on the Switch branches/tags button the top right of the page:

![Branch Naming](Chapter02/IMG/Branches-Names-Standards.png)

Branches with **stable/** prefix are still maintained. Non maintained OpenStack releases are named with branches with **unmaintained/** prefix.

Make sure to use the latest `stable` release to avoid compatibilty issues and potential bugs of new services.



> [!IMPORTANT]
> OpenStack Kolla-Ansible  code is still maintained project by OpenStack community and actively being developed. Upon a new OpenStack release, features or services can be introduced as well as deprecated and removed. For the best usage of the code in this book, it is highly recommended to have high level understanding of the Kolla-Ansible code structure that is described in different sections of the book.  



### OpenStack Deployment Approach 

The book presents different layouts and configurations for each chapter that takes into account the following approach:

- Each chapter introduces different changes that can be deployed separately

- Starting the OpenStack deployment with test environment that  run all services in one single host and does not require a CI/CD integration*

- Production environment would require multi nodes setup. Code changes must be maintained in central repository. A CI/CD pipeline must be built to faciliate OpenStack deployement, testing and capturing faster changes in separate environment 

- Additional services can be disabled when moving to the next chapter


Each Chapter requiring code has its guidelines for a specific OpenStack service deployment that can be found as the following:

- [Chapter02](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter02)
- [Chapter03](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter03)
- [Chapter04](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter04)
- [Chapter05](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter05)
- [Chapter06](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter06)
- [Chapter07](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter07)
- [Chapter08](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter08)
- [Chapter09](https://github.com/PacktPublishing/Mastering-OpenStack-Third-Edition/tree/main/Chapter09)





> [!IMPORTANT]
> The deployment in any environment will depend on existing resources. The Deployment in production environment would require more resources to minimum setup of core OpenStack services including compure, storage, network and memory hardware resources.


## Authors

Omar Khedher



## Resources

* Kolla-Ansbile community project: 
    * GitHub [repostority](https://github.com/openstack/kolla-ansible).
    * Documentation [Latest](https://docs.openstack.org/kolla-ansible/latest/).

* OpenStack Releases Notes:
    * Services [Projects List and Versions](https://releases.openstack.org/dalmatian/index.html).