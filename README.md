# OpenContrail-Kolla
OpenContrail Containers with Openstack Kolla

- [Introduction](#Introduction)
- [Requirements](#Requirements)
- [Components](#Requirements)
- [Configuration](#Configuration)


## Introduction

Opencontrail 4.0 introduce a containerized Contrail architecture where users can run all the Contrail components (control/config, analytics, analyticsdb and vRouter-agent) as Docker containers. This document enables users to install OpenContrail with Openstack-Kolla. Contrail-ansible provisions contrail containers and kolla-ansible provisions Openstack-Kolla (openstack containers), kolla-ansible is provided with the required parameters to use customized nova-compute and neutron containers with required changes to enable contrail functionalities.

All the Contrail containers are available publicly through Docker hub.

## Requirements

* Users can use a single server or a VM. Multinode cluster can be installed with the instructions mentioned in the installation section of the document.
* Ubuntu 16.04.2 LTS, Kernel: 4.4.0-62-generic
* The host VM/Machine must satisfy the following minimum requirements
  * 2 network interfaces (primary network interface with IP will be used as contrail-vrouter interface and openstack services interface, the second interface can be without IP address and is mostly unusable)
  * 32GB main memory (to enable considrable usage of the cluster)
  * 150GB disk space
* Root access to the machine is required

## Components

* contrail-ansible:
  * Playbooks to install Contrail-Controller, Contrail-Analytics, Contrail-AnalyticsDB
  * This installs Contrail-Vrouter-Agent container which pushes required configuration to the kernel and enable vRouter
  * Users can provide the configuration based on their environment in all.yml and hosts file

* kolla-ansible:
  * Playbooks to install all Openstack services as containers
  * Required parameters to use customized neutron and nova containers from Contrail repo are set in globals.yml
  * Users can use all-in-one inventory file to provision a single node Contrail and Openstack cluster. Users can use Multinode inventory file with required configuration to install a multinode cluster

## Configuration

Users can deploy Openstack using openstack-kolla first, followed by contrail-ansible. The configuration required is as follows:

* openstack-kolla:

  * OpenContrail-Kolla/kolla-ansible/etc/kolla/globals.yml
  
    ```
    # This should be the primary interface with IP address configured.
    network_interface: "ens3"
    ```
    This interface is what all the api services will be bound to by default. The same interface will be also used for contrail-vRouter physical interface.
    
    ```
    # This should be the second interface without IP address and mostly dormant and unusable.
    neutron_external_interface: "ens4"
    ```
    This is the raw interface given to neutron as its external network port. Even though an IP address can exist on this interface, it will be unusable in most configuration.
    
    ```
    # This should be the management address on the host, ens3 IP address which is the network_interface mentioned above.
    contrail_api_interface_address: "10.87.1.49"
    ```
    
  * OpenContrail-Kolla/kolla-ansible/etc/kolla/passwords.yml
  
    All the passwords in the configuration are set to "contrail1". If users intend to change the passwords the file can be updated.
    
  * OpenContrail-Kolla/kolla-ansible/ansible/inventory/all-in-one
  
    all-in-one: all the configuration can be left unchanged in case of a single node install
    
    ```
    [control]
    localhost       ansible_connection=local

    [network]
    localhost       ansible_connection=local

    [compute]
    localhost       ansible_connection=local

    [storage]
    localhost       ansible_connection=local

    [monitoring]
    localhost       ansible_connection=local

    [deployment]
    localhost       ansible_connection=local
    ```
    
  * OpenContrail-Kolla/kolla-ansible/ansible/inventory/multinode
  
    multinode: users can use this configuration file to install multinode-cluster
    
    ```
    [control]
    # These hostname must be resolvable from your deployment host
    1ocata

    [network]
    ocata1

    [compute]
    ocata2

    [monitoring]
    ocata1

    [storage]
    ocata1
    
    ```

* contrail-ansible:

  * OpenContrail-Kolla/contrail-ansible/playbooks/inventory/my-inventory/hosts
  
    Change the management IP address in the fields below. In case of a all-in-one cluster all the IP addresses should be the same. For multinode cluster multiple IP addresses can be provided in all the fields as required.  
    ```
    # Enable contrail-repo when required - this will start a contrail apt or yum repo container on specified node
    # This repo will be used by other nodes on installing any packages in the node
    # setting up contrail-cni need this repo enabled
    # NOTE: Repo is required only for mesos and nested mode kubernetes
    ;[contrail-repo]
    ;192.168.0.24

    [contrail-controllers]
    10.87.1.49

    [contrail-analyticsdb]
    10.87.1.49

    [contrail-analytics]
    10.87.1.49

    [contrail-compute]
    10.87.1.49

     ##
    # Only enable if you setup with openstack (when cloud_orchestrator is openstack)
    ##
    [openstack-controllers]
    10.87.1.49
    ```
 
  * OpenContrail-Kolla/contrail-ansible/playbooks/inventory/my-inventory/group_vars/all.yml
  
    Change the external_rabbitmq_servers to the host_ip, the same IP address used in the configuration above. If the user changes the default password the same should be updated in the rabbitmq_config below.
    ```
    # global_config:
    global_config: { external_rabbitmq_servers: 10.87.1.49 }
    rabbitmq_config: { user: openstack, password: contrail1 }
    ```
    Change the keystone_ip in keystone_config below, this should be the same address as mentioned above in the rabbitmq configuration 
    ```
    keystone_config: {ip: 10.87.1.49, admin_password: contrail1, auth_protocol: http}
    ```
    The primary interface with IP address on the host. THis should be the same interfaces mentioned in 'network_interface' section in globals.yml in the openstack_kolla configuration.
    ```
    # vrouter physical interface
    vrouter_physical_interface: ens3
    ```

## Provisioning

#### Packages and Dependencies

Install the following dependencies on the host:

```
apt-get update
apt-get install python-pip sshpass python-oslo-config python-dev libffi-dev gcc libssl-dev qemu-kvm
pip install -U pip
pip install -U ansible    #Upgrades pip to the latest version
pip install  pyOpenSSL==16.2.0
```

Install latest Docker on the host. The installation scripts installs older version of Docker by default.

```
curl -sSL https://get.docker.io | bash
```

#### Installation

Follow the sequence of installation steps:

* kolla-ansible

  * OpenContrail-Kolla/kolla-ansible/ansible/
  
   Bootstrap Host
   ```
    ansible-playbook -i inventory/all-in-one -e @../etc/kolla/globals.yml -e @../etc/kolla/passwords.yml -e      action=bootstrap-servers kolla-host.yml
    ```
   Deploy Openstack Containers-Kolla
   ```
   ansible-playbook -vvv -i inventory/all-in-one -e @../etc/kolla/globals.yml -e @../etc/kolla/passwords.yml -e action=deploy site.yml
   ```

* contrail-ansible

  * Copy keys for password less access and disable host_checking
  
    Generate keys:
    
    ```
    ssh-keygen -t rsa
    ```
    
    copy keys:
    
    ```
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@127.0.0.1
    ```

  * OpenContrail-Kolla/contrail-ansible/playbooks/
  
  Install Contrail Containers
  ```
  ansible-playbook  -i inventory/my-inventory site.yml
  ```
  
  


    
    

  
  

  







































