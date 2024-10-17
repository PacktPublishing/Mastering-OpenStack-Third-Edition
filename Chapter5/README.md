# Chapter 5
OpenStack Storage – Block, Object, and File Shares

## Description

The Chapter initiates the deployment of an OpenStack environment in different steps:
1. Multi Node Setup
2. Create a private repository to host the infrastructure code for future collaboration and maintenance



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


Example used from production environment, can use any compute node with other capacities example :




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


## Manila user Guide: Create a share in share servers mode (https://docs.openstack.org/manila/latest/admin/shared-file-systems-crud-share.html#create-a-share-in-share-servers-mode)


  
manila can support two modes, with and without the handling of share servers. The mode depends on driver support.

Mode 1: The back-end is a generic driver, driver_handles_share_servers = False (DHSS is disabled). The following example creates a VM using manila share image.





1. Install Manila client:
# apt install python3-manilaclient

2. By default Manila uses instance flavor id 100 for its file systems. For Manila to work, either create a new nova flavor with id 100 (use nova flavor-create) or change service_instance_flavor_id to use one of the default nova flavor ids. Ex: service_instance_flavor_id = 2 to use nova default flavor m1.small.

$ openstack flavor create --private m1.small --id 100 --ram 512 --disk 0 --vcpus 1

+----------------------------+----------+
| Field                      | Value    |
+----------------------------+----------+
| OS-FLV-DISABLED:disabled   | False    |
| OS-FLV-EXT-DATA:ephemeral  | 0        |
| description                | None     |
| disk                       | 0        |
| id                         | 100      |
| name                       | m1.small |
| os-flavor-access:is_public | False    |
| properties                 |          |
| ram                        | 512      |
| rxtx_factor                | 1.0      |
| swap                       | 0        |
| vcpus                      | 1        |
+----------------------------+----------+

3. Verify operation of the Shared File Systems service. List service components to verify successful launch of each process:

$ manila service-list
+----+------------------+--------------------------------+------+---------+-------+----------------------------+
| Id | Binary           | Host                           | Zone | Status  | State | Updated_at                 |
+----+------------------+--------------------------------+------+---------+-------+----------------------------+
| 1  | manila-data      | ubuntu2204.localdomain         | nova | enabled | up    | 2024-10-17T11:04:43.131783 |
| 2  | manila-scheduler | ubuntu2204.localdomain         | nova | enabled | up    | 2024-10-17T11:04:51.090763 |
| 3  | manila-share     | ubuntu2204.localdomain@generic | nova | enabled | up    | 2024-10-17T11:04:52.419310 |
+----+------------------+--------------------------------+------+---------+-------+----------------------------+

4.  Check share types that exist:

$ manila type-list
+--------------------------------------+--------------------+------------+------------+-------------------------------------+----------------------+-------------+
| ID                                   | Name               | visibility | is_default | required_extra_specs                | optional_extra_specs | Description |
+--------------------------------------+--------------------+------------+------------+-------------------------------------+----------------------+-------------+
| 748e6085-885c-4e16-9c38-56ce3cbf0cf0 | default_share_type | public     | -          | driver_handles_share_servers : True |                      | None        |
+--------------------------------------+--------------------+------------+------------+-------------------------------------+----------------------+-------------+



4. Before being able to create a share, the manila with the generic driver and the DHSS mode enabled requires the definition of at least an image, a network and a share-network for being used to create a share server. For that back end configuration, the share server is an instance where NFS/CIFS shares are served.

Create a default share type before running manila-share service:

$ manila type-create default_share_type True

+----------------------+--------------------------------------+
| Property             | Value                                |
+----------------------+--------------------------------------+
| ID                   | 748e6085-885c-4e16-9c38-56ce3cbf0cf0 |
| Name                 | default_share_type                   |
| Visibility           | public                               |
| is_default           | -                                    |
| required_extra_specs | driver_handles_share_servers : True  |
| optional_extra_specs |                                      |
| Description          | None                                 |
+----------------------+--------------------------------------+

5. Download a Manila share server image 

$ wget https://tarballs.opendev.org/openstack/manila-image-elements/images/manila-service-image-master.qcow2



6. Create a manila share server image to the Image service:

$ openstack image create   \
  --file manila-service-image-master.qcow2 \
  --disk-format qcow2 --container-format bare \
  --public --progress manila-service-image

[==========>                   ] 35%

+------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field            | Value                                                                                                                                                    |
+------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| container_format | bare                                                                                                                                                     |
| created_at       | 2024-10-17T11:16:58Z                                                                                                                                     |
| disk_format      | qcow2                                                                                                                                                    |
| file             | /v2/images/9be4165a-45a7-4d55-b2ef-65ed8d243adc/file                                                                                                     |
| id               | 9be4165a-45a7-4d55-b2ef-65ed8d243adc                                                                                                                     |
| min_disk         | 0                                                                                                                                                        |
| min_ram          | 0                                                                                                                                                        |
| name             | manila-service-image                                                                                                                                     |
| owner            | aa7e03d921cb4aec85e7c086abbfb99f                                                                                                                         |
| properties       | os_hidden='False', owner_specified.openstack.md5='', owner_specified.openstack.object='images/manila-service-image', owner_specified.openstack.sha256='' |
| protected        | False                                                                                                                                                    |
| schema           | /v2/schemas/image                                                                                                                                        |
| status           | queued                                                                                                                                                   |
| tags             |                                                                                                                                                          |
| updated_at       | 2024-10-17T11:16:58Z                                                                                                                                     |
| visibility       | public                                                                                                                                                   |
+------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+



7. Verify uploaded image:

$ openstack image list

+--------------------------------------+----------------------+--------+
| ID                                   | Name                 | Status |
+--------------------------------------+----------------------+--------+
| 9be4165a-45a7-4d55-b2ef-65ed8d243adc | manila-service-image | active |
+--------------------------------------+----------------------+--------+

8. List available networks to get id and subnets of the private network if they do not exist.
Check network list:
openstack network list if not create a new private network:

$ openstack network create private
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2024-10-17T11:30:05Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 6de0f4c1-ccf1-4188-874d-25fec48036ff |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | private                              |
| port_security_enabled     | True                                 |
| project_id                | aa7e03d921cb4aec85e7c086abbfb99f     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 641                                  |
| qos_policy_id             | None                                 |
| revision_number           | 1                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2024-10-17T11:30:05Z                 |
+---------------------------+--------------------------------------+



9. Create subnet private range:

$ openstack subnet create subnet1 --network private --subnet-range 192.0.2.0/24

+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| allocation_pools     | 192.0.2.2-192.0.2.254                |
| cidr                 | 192.0.2.0/24                         |
| created_at           | 2024-10-17T11:36:26Z                 |
| description          |                                      |
| dns_nameservers      |                                      |
| dns_publish_fixed_ip | None                                 |
| enable_dhcp          | True                                 |
| gateway_ip           | 192.0.2.1                            |
| host_routes          |                                      |
| id                   | 15888836-c5b6-430b-b629-c588fd305fba |
| ip_version           | 4                                    |
| ipv6_address_mode    | None                                 |
| ipv6_ra_mode         | None                                 |
| name                 | subnet1                              |
| network_id           | 6de0f4c1-ccf1-4188-874d-25fec48036ff |
| project_id           | aa7e03d921cb4aec85e7c086abbfb99f     |
| revision_number      | 0                                    |
| router:external      | False                                |
| segment_id           | None                                 |
| service_types        |                                      |
| subnetpool_id        | None                                 |
| tags                 |                                      |
| updated_at           | 2024-10-17T11:36:26Z                 |
+----------------------+--------------------------------------+



10. Create share network using the ID of the created network as well as the subnet:

$ manila share-network-create --name share-network1 \
  --neutron-net-id 6de0f4c1-ccf1-4188-874d-25fec48036ff \
  --neutron-subnet-id 15888836-c5b6-430b-b629-c588fd305fba

+---------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Property                        | Value                                                                                                                                                                                                                                                                                                                                                                             |
+---------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                              | d0d0bc63-fe77-401d-8037-f4a393382d78                                                                                                                                                                                                                                                                                                                                              |
| name                            | share-network1                                                                                                                                                                                                                                                                                                                                                                    |
| project_id                      | aa7e03d921cb4aec85e7c086abbfb99f                                                                                                                                                                                                                                                                                                                                                  |
| created_at                      | 2024-10-17T11:38:41.258565                                                                                                                                                                                                                                                                                                                                                        |
| updated_at                      | None                                                                                                                                                                                                                                                                                                                                                                              |
| description                     | None                                                                                                                                                                                                                                                                                                                                                                              |
| share_network_subnets           | [{'id': 'b92482ca-2b50-429d-a1ee-82c6d7b00d81', 'availability_zone': None, 'created_at': '2024-10-17T11:38:41.552257', 'updated_at': None, 'segmentation_id': None, 'neutron_net_id': '6de0f4c1-ccf1-4188-874d-25fec48036ff', 'neutron_subnet_id': '15888836-c5b6-430b-b629-c588fd305fba', 'ip_version': None, 'cidr': None, 'network_type': None, 'mtu': None, 'gateway': None}] |
| status                          | active                                                                                                                                                                                                                                                                                                                                                                            |
| security_service_update_support | True                                                                                                                                                                                                                                                                                                                                                                              |
+---------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

11. Verify the shared network:

manila share-network-list
+--------------------------------------+----------------+
| id                                   | name           |
+--------------------------------------+----------------+
| d0d0bc63-fe77-401d-8037-f4a393382d78 | share-network1 |
+--------------------------------------+----------------+


12. Create share:

$ manila create NFS 1 --name PP_SHARE1 --share-network share-network1 --share-type default_share_type
+---------------------------------------+--------------------------------------+
| Property                              | Value                                |
+---------------------------------------+--------------------------------------+
| id                                    | edf2dc8c-b6d7-4337-a65c-2d7f2ab610c0 |
| size                                  | 1                                    |
| availability_zone                     | None                                 |
| created_at                            | 2024-10-17T11:44:22.192810           |
| status                                | creating                             |
| name                                  | PP_SHARE1                            |
| description                           | None                                 |
| project_id                            | aa7e03d921cb4aec85e7c086abbfb99f     |
| snapshot_id                           | None                                 |
| share_network_id                      | d0d0bc63-fe77-401d-8037-f4a393382d78 |
| share_proto                           | NFS                                  |
| metadata                              | {}                                   |
| share_type                            | 748e6085-885c-4e16-9c38-56ce3cbf0cf0 |
| is_public                             | False                                |
| snapshot_support                      | False                                |
| task_state                            | None                                 |
| share_type_name                       | default_share_type                   |
| access_rules_status                   | active                               |
| replication_type                      | None                                 |
| has_replicas                          | False                                |
| user_id                               | accf193278c646a9b78e3034cd09e96e     |
| create_share_from_snapshot_support    | False                                |
| revert_to_snapshot_support            | False                                |
| share_group_id                        | None                                 |
| source_share_group_snapshot_member_id | None                                 |
| mount_snapshot_support                | False                                |
| progress                              | None                                 |
| is_soft_deleted                       | False                                |
| scheduled_to_be_deleted_at            | None                                 |
| share_server_id                       | None                                 |
| host                                  |                                      |
+---------------------------------------+--------------------------------------+