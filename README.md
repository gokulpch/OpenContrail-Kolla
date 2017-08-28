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
    

* contrail-ansible:


  
  

  







































