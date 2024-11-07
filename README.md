# Mastering OpenStack

<a href="https://www.packtpub.com/en-us/product/mastering-openstack-9781835468913"><img src="https://content.packt.com/B21716/cover_image_small.jpg" alt="no-image" height="256px" align="right"></a>

This is the code repository for [Mastering OpenStack](https://www.packtpub.com/en-us/product/mastering-openstack-9781835468913), published by Packt.

**Implement the latest techniques for designing and deploying an operational, production-ready private cloud**

## What is this book about?
This edition guides you through the latest updates in the OpenStack ecosystem, offering insights into cloud design with new services and operational best practices. This book also explores the hybrid cloud era with OpenStack and design patterns.

This book covers the following exciting features:
* Explore the latest design patterns in the OpenStack ecosystem
* Implement DevSecOps practices for agile and secure deployment management
* Ensure resilience, fault tolerance, and performance in your cloud setup
* Stay up to date with OpenStack networking and storage advancements
* Master operational best practices for managing a large-scale cloud setup
* Discover logging and monitoring options for your cloud
* Get acquainted with new services such as SDN and containers
* Understand how to extend OpenStack’s capabilities through a hybrid model

If you feel this book is for you, get your [copy](https://www.amazon.com/Mastering-OpenStack-techniques-operational-production-ready/dp/1835468918) today!

<a href="https://www.packtpub.com/?utm_source=github&utm_medium=banner&utm_campaign=GitHubBanner"><img src="https://raw.githubusercontent.com/PacktPublishing/GitHub/master/GitHub.png" 
alt="https://www.packtpub.com/" border="5" /></a>

## Instructions and Navigations
All of the code is organized into folders. For example, Chapter02.

The code will look like the following:
```
…
[magnum:children]
control
[magnum-api:children]
magnum
[magnum-conductor:children]
magnum
…
```
**Following is what you need for this book:**
This book is for OpenStack administrators, cloud and enterprise architects, and system and DevOps engineers looking to launch a private cloud with OpenStack. If you’re a cloud advisor, consultant, or evangelist, you’ll benefit from the expert insights, and if you’re a software developer or system operator aiming to accelerate your development cycle and agility, this book will help you bridge the gap between these roles. Basic knowledge of OpenStack, along with a prior understanding of systems, virtualization, and networking, is recommended.

With the following software and hardware list you can run all code files present in the book (Chapter 1-11).
### Software and Hardware List
| Software required | OS required |
| ------------------------------------ | ----------------------------------- |
| OpenStack (Antelope or later releases), AWS account | - |
| Python, YAML, JSON | - |
| Ansible, Kolla, Jenkins, Docker, Kubernetes | - |
| Ceph, Canonical Juju, KubeFed | - |

<a href="https://www.packtpub.com/?utm_source=github&utm_medium=banner&utm_campaign=GitHubBanner"><img src="https://raw.githubusercontent.com/PacktPublishing/GitHub/master/GitHub.png" 
alt="https://www.packtpub.com/" border="5" /></a>

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


## Resources

* Kolla-Ansbile community project: 
    * GitHub [repostority](https://github.com/openstack/kolla-ansible).
    * Documentation [Latest](https://docs.openstack.org/kolla-ansible/latest/).

* OpenStack Releases Notes:
    * Services [Projects List and Versions](https://releases.openstack.org/dalmatian/index.html).
    



<a href="https://www.packtpub.com/?utm_source=github&utm_medium=banner&utm_campaign=GitHubBanner"><img src="https://raw.githubusercontent.com/PacktPublishing/GitHub/master/GitHub.png" 
alt="https://www.packtpub.com/" border="5" /></a>


### Related products
* The Self-Taught Cloud Computing Engineer [[Packt]](https://www.packtpub.com/en-US/product/the-self-taught-cloud-computing-engineer-9781805123705) [[Amazon]](https://www.amazon.com/dp/180512370X)

* Enterprise-Grade Hybrid and Multi-Cloud Strategies [[Packt]](https://www.packtpub.com/en-us/product/enterprise-grade-hybrid-and-multi-cloud-strategies-9781804615119) [[Amazon]](https://www.amazon.com/dp/1804615110)

<a href="https://www.packtpub.com/?utm_source=github&utm_medium=banner&utm_campaign=GitHubBanner"><img src="https://raw.githubusercontent.com/PacktPublishing/GitHub/master/GitHub.png" 
alt="https://www.packtpub.com/" border="5" /></a>

## Get to Know the Author
**Omar Khedher**
 is a cloud architect with more than 12 years of experience in the cloud computing market. He has been involved in several private and public cloud projects, including OpenStack, AWS, and Azure. He has also authored several other books, including the first and second editions of Mastering OpenStack, OpenStack Sahara Essentials, and Extending OpenStack. Omar has a few academic publications about cloud performance. As a part of the Cloudreach team, an international cloud consulting company, he helps customers to design, advises on, and leads projects targeting cloud migration and innovation.

## Other books by the author
* Extending OpenStack [[Packt]](https://www.packtpub.com/en-us/product/extending-openstack-9781786465535) [[Amazon]](https://www.amazon.com/Extending-OpenStack-containerization-deployment-architecting/dp/1786465531)

* OpenStack Sahara Essentials [[Packt]](https://www.packtpub.com/en-us/product/openstack-sahara-essentials-9781785885969) [[Amazon]](https://www.amazon.com/OpenStack-Sahara-Essentials-Omar-Khedher-ebook/dp/B01CGKAJES)


