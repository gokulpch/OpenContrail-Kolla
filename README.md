# OpenContrail-Kolla
OpenContrail Containers with Openstack Kolla

- [Introduction](#Introduction)
- [Requirements](#Requirements)

## Introduction

Opencontrail 4.0 introduce a containerized Contrail architecture where users can run all the Contrail components (control/config, analytics, analyticsdb and vRouter-agent) as Docker containers. This document enables users to install OpenContrail with Openstack-Kolla. The integral components of the procedure are: contrail-ansible to provision contrai containers and kolla-ansible to provision openstack-kolla, kolla-ansible is provided with the required parameters to use nova-compute and neutron containers with required changes to enable contrail functionalities.

## Requirements

* Users can use a single server or a VM. Multinode cluster can be installed with the instructions mentioned in the installation section of the document.
* Ubuntu 16.04.2 LTS, Kernel: 4.4.0-62-generic
* The host VM/Machine must satisfy the following minimum requirements
            * 2 network interfaces (primary network interface with IP will be used as contrail-vrouter interface and openstack services interface, the second interface can be without IP address and is mostly unusable)
            * 32GB main memory (to enable considrable usage of the cluster)
            * 150GB disk space







































