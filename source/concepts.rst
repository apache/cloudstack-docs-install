.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information#
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at
   http://www.apache.org/licenses/LICENSE-2.0
   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.

Concepts
========

What Is CloudStack?
-------------------

CloudStack is an open source software platform that pools computing
resources to build public, private, and hybrid Infrastructure as a
Service (IaaS) clouds. CloudStack manages the network, storage, and
compute nodes that make up a cloud infrastructure. Use CloudStack to
deploy, manage, and configure cloud computing environments.

Typical users are service providers and enterprises. With CloudStack,
you can:

-  

   Set up an on-demand, elastic cloud computing service. Service
   providers can sell self service virtual machine instances, storage
   volumes, and networking configurations over the Internet.

-  

   Set up an on-premise private cloud for use by employees. Rather than
   managing virtual machines in the same way as physical machines, with
   CloudStack an enterprise can offer self-service virtual machines to
   users without involving IT departments.

|1000-foot-view.png: Overview of CloudStack|

What Can CloudStack Do?
-----------------------

**Multiple Hypervisor Support**

CloudStack works with a variety of hypervisors, and a single cloud
deployment can contain multiple hypervisor implementations. The current
release of CloudStack supports pre-packaged enterprise solutions like
Citrix XenServer and VMware vSphere, as well as KVM or Xen running on
Ubuntu or CentOS.

**Massively Scalable Infrastructure Management**

CloudStack can manage tens of thousands of servers installed in multiple
geographically distributed datacenters. The centralized management
server scales linearly, eliminating the need for intermediate
cluster-level management servers. No single component failure can cause
a cloud-wide outage. Periodic maintenance of the management server can
be performed without affecting the functioning of virtual machines
running in the cloud.

**Automatic Configuration Management**

CloudStack automatically configures each guest virtual machine’s
networking and storage settings.

CloudStack internally manages a pool of virtual appliances to support
the cloud itself. These appliances offer services such as firewalling,
routing, DHCP, VPN access, console proxy, storage access, and storage
replication. The extensive use of virtual appliances simplifies the
installation, configuration, and ongoing management of a cloud
deployment.

**Graphical User Interface**

CloudStack offers an administrator's Web interface, used for
provisioning and managing the cloud, as well as an end-user's Web
interface, used for running VMs and managing VM templates. The UI can be
customized to reflect the desired service provider or enterprise look
and feel.

**API and Extensibility**

CloudStack provides an API that gives programmatic access to all the
management features available in the UI. The API is maintained and
documented. This API enables the creation of command line tools and new
user interfaces to suit particular needs. See the Developer’s Guide and
API Reference, both available at `Apache CloudStack
Guides <http://cloudstack.apache.org/docs/en-US/index.html>`__ and
`Apache CloudStack API
Reference <http://cloudstack.apache.org/docs/api/index.html>`__
respectively.

The CloudStack pluggable allocation architecture allows the creation of
new types of allocators for the selection of storage and Hosts. See the
Allocator Implementation Guide
(`http://docs.cloudstack.org/CloudStack\_Documentation/Allocator\_Implementation\_Guide <http://docs.cloudstack.org/CloudStack_Documentation/Allocator_Implementation_Guide>`__).

**High Availability**

CloudStack has a number of features to increase the availability of the
system. The Management Server itself may be deployed in a multi-node
installation where the servers are load balanced. MySQL may be
configured to use replication to provide for a manual failover in the
event of database loss. For the hosts, CloudStack supports NIC bonding
and the use of separate networks for storage as well as iSCSI Multipath.

Deployment Architecture Overview
--------------------------------

A CloudStack installation consists of two parts: the Management Server
and the cloud infrastructure that it manages. When you set up and manage
a CloudStack cloud, you provision resources such as hosts, storage
devices, and IP addresses into the Management Server, and the Management
Server manages those resources.

The minimum production installation consists of one machine running the
CloudStack Management Server and another machine to act as the cloud
infrastructure (in this case, a very simple infrastructure consisting of
one host running hypervisor software). In its smallest deployment, a
single machine can act as both the Management Server and the hypervisor
host (using the KVM hypervisor).

|basic-deployment.png: Basic two-machine deployment|

A more full-featured installation consists of a highly-available
multi-node Management Server installation and up to tens of thousands of
hosts using any of several advanced networking setups. For information
about deployment options, see the "Choosing a Deployment Architecture"
section of the CloudStack Installation Guide.

Management Server Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~

The Management Server is the CloudStack software that manages cloud
resources. By interacting with the Management Server through its UI or
API, you can configure and manage your cloud infrastructure.

The Management Server runs on a dedicated server or VM. It controls
allocation of virtual machines to hosts and assigns storage and IP
addresses to the virtual machine instances. The Management Server runs
in a Tomcat container and requires a MySQL database for persistence.

The machine must meet the system requirements described in System
Requirements.

The Management Server:

-  

   Provides the web user interface for the administrator and a reference
   user interface for end users.

-  

   Provides the APIs for CloudStack.

-  

   Manages the assignment of guest VMs to particular hosts.

-  

   Manages the assignment of public and private IP addresses to
   particular accounts.

-  

   Manages the allocation of storage to guests as virtual disks.

-  

   Manages snapshots, templates, and ISO images, possibly replicating
   them across data centers.

-  

   Provides a single point of configuration for the cloud.

Cloud Infrastructure Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Management Server manages one or more zones (typically, datacenters)
containing host computers where guest virtual machines will run. The
cloud infrastructure is organized as follows:

-  

   Zone: Typically, a zone is equivalent to a single datacenter. A zone
   consists of one or more pods and secondary storage.

-  

   Pod: A pod is usually one rack of hardware that includes a layer-2
   switch and one or more clusters.

-  

   Cluster: A cluster consists of one or more hosts and primary storage.

-  

   Host: A single compute node within a cluster. The hosts are where the
   actual cloud services run in the form of guest virtual machines.

-  

   Primary storage is associated with a cluster, and it stores the disk
   volumes for all the VMs running on hosts in that cluster.

-  

   Secondary storage is associated with a zone, and it stores templates,
   ISO images, and disk volume snapshots.

|infrastructure_overview.png: Nested organization of a zone|

**More Information**

For more information, see documentation on cloud infrastructure
concepts.

Networking Overview
~~~~~~~~~~~~~~~~~~~

CloudStack offers two types of networking scenario:

-  

   Basic. For AWS-style networking. Provides a single network where
   guest isolation can be provided through layer-3 means such as
   security groups (IP address source filtering).

-  

   Advanced. For more sophisticated network topologies. This network
   model provides the most flexibility in defining guest networks.

.. |1000-foot-view.png: Overview of CloudStack| image:: ./_static/images/1000-foot-view.png
.. |basic-deployment.png: Basic two-machine deployment| image:: ./_static/images/basic-deployment.png
.. |infrastructure_overview.png: Nested organization of a zone| image:: ./_static/images/infrastructure-overview.png
.. |region-overview.png: Nested structure of a region.| image:: ./_static/images/region-overview.png
.. |zone-overview.png: Nested structure of a simple zone.| image:: ./_static/images/zone-overview.png
.. |pod-overview.png: Nested structure of a simple pod| image:: ./_static/images/pod-overview.png
.. |cluster-overview.png: Structure of a simple cluster| image:: ./_static/images/cluster-overview.png
.. |installation-complete.png: Finished installs with single Management Server and multiple Management Servers| image:: ./_static/images/installation-complete.png
.. |change-password.png: button to change a user's password| image:: ./_static/images/change-password.png
.. |provisioning-overview.png: Conceptual overview of a basic deployment| image:: ./_static/images/provisioning-overview.png
.. |vsphereclient.png: vSphere client| image:: ./_static/images/vsphere-client.png
.. |addcluster.png: add a cluster| image:: ./_static/images/add-cluster.png
.. |ConsoleButton.png: button to launch a console| image:: ./_static/images/console-icon.png
.. |DeleteButton.png: button to delete dvSwitch| image:: ./_static/images/delete-button.png
.. |vds-name.png: Name of the dvSwitch as specified in the vCenter.| image:: ./_static/images/vds-name.png
.. |traffic-type.png: virtual switch type| image:: ./_static/images/traffic-type.png
.. |dvSwitchConfig.png: Configuring dvSwitch| image:: ./_static/images/dvSwitch-config.png
.. |Small-Scale Deployment| image:: ./_static/images/small-scale-deployment.png
.. |Large-Scale Redundant Setup| image:: ./_static/images/large-scale-redundant-setup.png
.. |Multi-Node Management Server| image:: ./_static/images/multi-node-management-server.png
.. |Example Of A Multi-Site Deployment| image:: ./_static/images/multi-site-deployment.png
.. |Separate Storage Network| image:: ./_static/images/separate-storage-network.png
.. |NIC Bonding And Multipath I/O| image:: ./_static/images/nic-bonding-and-multipath-io.png
.. |Use the GUI to set the configuration variable to true| image:: ./_static/images/ec2-s3-configuration.png
.. |Use the GUI to set the name of a compute service offering to an EC2 instance type API name.| image:: ./_static/images/compute-service-offerings.png
.. |parallel-mode.png: adding a firewall and load balancer in parallel mode.| image:: ./_static/images/parallel-mode.png
.. |guest-traffic-setup.png: Depicts a guest traffic setup| image:: ./_static/images/guest-traffic-setup.png
.. |networksinglepod.png: diagram showing logical view of network in a pod| image:: ./_static/images/network-singlepod.png
.. |networksetupzone.png: Depicts network setup in a single zone| image:: ./_static/images/network-setup-zone.png
.. |addguestnetwork.png: Add Guest network setup in a single zone| image:: ./_static/images/add-guest-network.png
.. |remove-nic.png: button to remove a NIC| image:: ./_static/images/remove-nic.png
.. |set-default-nic.png: button to set a NIC as default one.| image:: ./_static/images/set-default-nic.png
.. |EditButton.png: button to edit a network| image:: ./_static/images/edit-icon.png
.. |edit-icon.png: button to edit a network| image:: ./_static/images/edit-icon.png
.. |addAccount-icon.png: button to assign an IP range to an account.| image:: ./_static/images/addAccount-icon.png
.. |eip-ns-basiczone.png: Elastic IP in a NetScaler-enabled Basic Zone.| image:: ./_static/images/eip-ns-basiczone.png
.. |add-ip-range.png: adding an IP range to a network.| image:: ./_static/images/add-ip-range.png
.. |httpaccess.png: allows inbound HTTP access from anywhere| image:: ./_static/images/http-access.png
.. |autoscaleateconfig.png: Configuring AutoScale| image:: ./_static/images/autoscale-config.png
.. |EnableDisable.png: button to enable or disable AutoScale.| image:: ./_static/images/enable-disable-autoscale.png
.. |gslb.png: GSLB architecture| image:: ./_static/images/gslb.png
.. |gslb-add.png: adding a gslb rule| image:: ./_static/images/add-gslb.png
.. |ReleaseIPButton.png: button to release an IP| image:: ./_static/images/release-ip-icon.png
.. |EnableNATButton.png: button to enable NAT| image:: ./_static/images/enable-disable.png
.. |egress-firewall-rule.png: adding an egress firewall rule| image:: ./_static/images/egress-firewall-rule.png
.. |AttachDiskButton.png: button to attach a volume| image:: ./_static/images/vpn-icon.png
.. |vpn-icon.png: button to enable VPN| image:: ./_static/images/vpn-icon.png
.. |addvpncustomergateway.png: adding a customer gateway.| image:: ./_static/images/add-vpn-customer-gateway.png
.. |edit.png: button to edit a VPN customer gateway| image:: ./_static/images/edit-icon.png
.. |delete.png: button to remove a VPN customer gateway| image:: ./_static/images/delete-button.png
.. |createvpnconnection.png: creating a VPN connection to the customer gateway.| image:: ./_static/images/create-vpn-connection.png
.. |remove-vpn.png: button to remove a VPN connection| image:: ./_static/images/remove-vpn.png
.. |reset-vpn.png: button to reset a VPN connection| image:: ./_static/images/reset-vpn.png
.. |mutltier.png: a multi-tier setup.| image:: ./_static/images/multi-tier-app.png
.. |add-vpc.png: adding a vpc.| image:: ./_static/images/add-vpc.png
.. |add-tier.png: adding a tier to a vpc.| image:: ./_static/images/add-tier.png
.. |replace-acl-icon.png: button to replace an ACL list| image:: ./_static/images/replace-acl-icon.png
.. |add-new-gateway-vpc.png: adding a private gateway for the VPC.| image:: ./_static/images/add-new-gateway-vpc.png
.. |replace-acl-icon.png: button to replace the default ACL behaviour.| image:: ./_static/images/replace-acl-icon.png
.. |add-vm-vpc.png: adding a VM to a vpc.| image:: ./_static/images/add-vm-vpc.png
.. |addvm-tier-sharednw.png: adding a VM to a VPC tier and shared network.| image:: ./_static/images/addvm-tier-sharednw.png
.. |release-ip-icon.png: button to release an IP.| image:: ./_static/images/release-ip-icon.png
.. |enable-disable.png: button to enable Static NAT.| image:: ./_static/images/enable-disable.png
.. |select-vmstatic-nat.png: selecting a tier to apply staticNAT.| image:: ./_static/images/select-vm-staticnat-vpc.png
.. |vpc-lb.png: Configuring internal LB for VPC| image:: ./_static/images/vpc-lb.png
.. |del-tier.png: button to remove a tier| image:: ./_static/images/del-tier.png
.. |remove-vpc.png: button to remove a VPC| image:: ./_static/images/remove-vpc.png
.. |edit-icon.png: button to edit a VPC| image:: ./_static/images/edit-icon.png
.. |restart-vpc.png: button to restart a VPC| image:: ./_static/images/restart-vpc.png
