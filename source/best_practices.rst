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

Best Practices
==============

Deploying a cloud is challenging. There are many different technology
choices to make, and CloudStack is flexible enough in its configuration
that there are many possible ways to combine and configure the chosen
technology. This section contains suggestions and requirements about
cloud deployments.

These should be treated as suggestions and not absolutes. However, we do
encourage anyone planning to build a cloud outside of these guidelines
to seek guidance and advice on the project mailing lists.

Process Best Practices
----------------------

-  

   A staging system that models the production environment is strongly
   advised. It is critical if customizations have been applied to
   CloudStack.

-  

   Allow adequate time for installation, a beta, and learning the
   system. Installs with basic networking can be done in hours. Installs
   with advanced networking usually take several days for the first
   attempt, with complicated installations taking longer. For a full
   production system, allow at least 4-8 weeks for a beta to work
   through all of the integration issues. You can get help from fellow
   users on the cloudstack-users mailing list.

Setup Best Practices
--------------------

-  

   Each host should be configured to accept connections only from
   well-known entities such as the CloudStack Management Server or your
   network monitoring software.

-  

   Use multiple clusters per pod if you need to achieve a certain switch
   density.

-  

   Primary storage mountpoints or LUNs should not exceed 6 TB in size.
   It is better to have multiple smaller primary storage elements per
   cluster than one large one.

-  

   When exporting shares on primary storage, avoid data loss by
   restricting the range of IP addresses that can access the storage.
   See "Linux NFS on Local Disks and DAS" or "Linux NFS on iSCSI".

-  

   NIC bonding is straightforward to implement and provides increased
   reliability.

-  

   10G networks are generally recommended for storage access when larger
   servers that can support relatively more VMs are used.

-  

   Host capacity should generally be modeled in terms of RAM for the
   guests. Storage and CPU may be overprovisioned. RAM may not. RAM is
   usually the limiting factor in capacity designs.

-  

   (XenServer) Configure the XenServer dom0 settings to allocate more
   memory to dom0. This can enable XenServer to handle larger numbers of
   virtual machines. We recommend 2940 MB of RAM for XenServer dom0. For
   instructions on how to do this, see
   `http://support.citrix.com/article/CTX126531 <http://support.citrix.com/article/CTX126531>`__.
   The article refers to XenServer 5.6, but the same information applies
   to XenServer 6.0.

Maintenance Best Practices
--------------------------

-  

   Monitor host disk space. Many host failures occur because the host's
   root disk fills up from logs that were not rotated adequately.

-  

   Monitor the total number of VM instances in each cluster, and disable
   allocation to the cluster if the total is approaching the maximum
   that the hypervisor can handle. Be sure to leave a safety margin to
   allow for the possibility of one or more hosts failing, which would
   increase the VM load on the other hosts as the VMs are redeployed.
   Consult the documentation for your chosen hypervisor to find the
   maximum permitted number of VMs per host, then use CloudStack global
   configuration settings to set this as the default limit. Monitor the
   VM activity in each cluster and keep the total number of VMs below a
   safe level that allows for the occasional host failure. For example,
   if there are N hosts in the cluster, and you want to allow for one
   host in the cluster to be down at any given time, the total number of
   VM instances you can permit in the cluster is at most (N-1) \*
   (per-host-limit). Once a cluster reaches this number of VMs, use the
   CloudStack UI to disable allocation to the cluster.

.. warning:: The lack of up-do-date hotfixes can lead to data corruption and lost
VMs. Be sure all the hotfixes provided by the hypervisor vendor are applied.
Track the release of hypervisor patches through your hypervisor vendor’s
support channel, and apply patches as soon as possible after they are
released. CloudStack will not track or notify you of required hypervisor
patches. It is essential that your hosts are completely up to date with
the provided hypervisor patches. The hypervisor vendor is likely to
refuse to support any system that is not up to date with patches.


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
