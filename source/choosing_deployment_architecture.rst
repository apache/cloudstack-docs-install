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

Choosing a Deployment Architecture
==================================

The architecture used in a deployment will vary depending on the size
and purpose of the deployment. This section contains examples of
deployment architecture, including a small-scale deployment useful for
test and trial deployments and a fully-redundant large-scale setup for
production deployments.

Small-Scale Deployment
----------------------

|Small-Scale Deployment|

This diagram illustrates the network architecture of a small-scale
CloudStack deployment.

-  

   A firewall provides a connection to the Internet. The firewall is
   configured in NAT mode. The firewall forwards HTTP requests and API
   calls from the Internet to the Management Server. The Management
   Server resides on the management network.

-  

   A layer-2 switch connects all physical servers and storage.

-  

   A single NFS server functions as both the primary and secondary
   storage.

-  

   The Management Server is connected to the management network.

Large-Scale Redundant Setup
---------------------------

|Large-Scale Redundant Setup|

This diagram illustrates the network architecture of a large-scale
CloudStack deployment.

-  

   A layer-3 switching layer is at the core of the data center. A router
   redundancy protocol like VRRP should be deployed. Typically high-end
   core switches also include firewall modules. Separate firewall
   appliances may also be used if the layer-3 switch does not have
   integrated firewall capabilities. The firewalls are configured in NAT
   mode. The firewalls provide the following functions:

   -  

      Forwards HTTP requests and API calls from the Internet to the
      Management Server. The Management Server resides on the management
      network.

   -  

      When the cloud spans multiple zones, the firewalls should enable
      site-to-site VPN such that servers in different zones can directly
      reach each other.

-  

   A layer-2 access switch layer is established for each pod. Multiple
   switches can be stacked to increase port count. In either case,
   redundant pairs of layer-2 switches should be deployed.

-  

   The Management Server cluster (including front-end load balancers,
   Management Server nodes, and the MySQL database) is connected to the
   management network through a pair of load balancers.

-  

   Secondary storage servers are connected to the management network.

-  

   Each pod contains storage and computing servers. Each storage and
   computing server should have redundant NICs connected to separate
   layer-2 access switches.

Separate Storage Network
------------------------

In the large-scale redundant setup described in the previous section,
storage traffic can overload the management network. A separate storage
network is optional for deployments. Storage protocols such as iSCSI are
sensitive to network delays. A separate storage network ensures guest
network traffic contention does not impact storage performance.

Multi-Node Management Server
----------------------------

The CloudStack Management Server is deployed on one or more front-end
servers connected to a single MySQL database. Optionally a pair of
hardware load balancers distributes requests from the web. A backup
management server set may be deployed using MySQL replication at a
remote site to add DR capabilities.

|Multi-Node Management Server|

The administrator must decide the following.

-  

   Whether or not load balancers will be used.

-  

   How many Management Servers will be deployed.

-  

   Whether MySQL replication will be deployed to enable disaster
   recovery.

Multi-Site Deployment
---------------------

The CloudStack platform scales well into multiple sites through the use
of zones. The following diagram shows an example of a multi-site
deployment.

|Example Of A Multi-Site Deployment|

Data Center 1 houses the primary Management Server as well as zone 1.
The MySQL database is replicated in real time to the secondary
Management Server installation in Data Center 2.

|Separate Storage Network|

This diagram illustrates a setup with a separate storage network. Each
server has four NICs, two connected to pod-level network switches and
two connected to storage network switches.

There are two ways to configure the storage network:

-  

   Bonded NIC and redundant switches can be deployed for NFS. In NFS
   deployments, redundant switches and bonded NICs still result in one
   network (one CIDR block+ default gateway address).

-  

   iSCSI can take advantage of two separate storage networks (two CIDR
   blocks each with its own default gateway). Multipath iSCSI client can
   failover and load balance between separate storage networks.

|NIC Bonding And Multipath I/O|

This diagram illustrates the differences between NIC bonding and
Multipath I/O (MPIO). NIC bonding configuration involves only one
network. MPIO involves two separate networks.


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
