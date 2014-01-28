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

Configuration Parameters
========================

About Configuration Parameters
------------------------------

CloudStack provides a variety of settings you can use to set limits,
configure features, and enable or disable features in the cloud. Once
your Management Server is running, you might need to set some of these
configuration parameters, depending on what optional features you are
setting up. You can set default values at the global level, which will
be in effect throughout the cloud unless you override them at a lower
level. You can make local settings, which will override the global
configuration parameter values, at the level of an account, zone,
cluster, or primary storage.

The documentation for each CloudStack feature should direct you to the
names of the applicable parameters. The following table shows a few of
the more useful parameters.

Field

Value

management.network.cidr

A CIDR that describes the network that the management CIDRs reside on.
This variable must be set for deployments that use vSphere. It is
recommended to be set for other deployments as well. Example:
192.168.3.0/24.

xen.setup.multipath

For XenServer nodes, this is a true/false variable that instructs
CloudStack to enable iSCSI multipath on the XenServer Hosts when they
are added. This defaults to false. Set it to true if you would like
CloudStack to enable multipath.

If this is true for a NFS-based deployment multipath will still be
enabled on the XenServer host. However, this does not impact NFS
operation and is harmless.

secstorage.allowed.internal.sites

This is used to protect your internal network from rogue attempts to
download arbitrary files using the template download feature. This is a
comma-separated list of CIDRs. If a requested URL matches any of these
CIDRs the Secondary Storage VM will use the private network interface to
fetch the URL. Other URLs will go through the public interface. We
suggest you set this to 1 or 2 hardened internal machines where you keep
your templates. For example, set it to 192.168.1.66/32.

use.local.storage

Determines whether CloudStack will use storage that is local to the Host
for data disks, templates, and snapshots. By default CloudStack will not
use this storage. You should change this to true if you want to use
local storage and you understand the reliability and feature drawbacks
to choosing local storage.

host

This is the IP address of the Management Server. If you are using
multiple Management Servers you should enter a load balanced IP address
that is reachable via the private network.

default.page.size

Maximum number of items per page that can be returned by a CloudStack
API command. The limit applies at the cloud level and can vary from
cloud to cloud. You can override this with a lower value on a particular
API call by using the page and pagesize API command parameters. For more
information, see the Developer's Guide. Default: 500.

ha.tag

The label you want to use throughout the cloud to designate certain
hosts as dedicated HA hosts. These hosts will be used only for
HA-enabled VMs that are restarting due to the failure of another host.
For example, you could set this to ha\_host. Specify the ha.tag value as
a host tag when you add a new host to the cloud.

vmware.vcenter.session.timeout

Determines the vCenter session timeout value by using this parameter.
The default value is 20 minutes. Increase the timeout value to avoid
timeout errors in VMware deployments because certain VMware operations
take more than 20 minutes.

Setting Global Configuration Parameters
---------------------------------------

Use the following steps to set global configuration parameters. These
values will be the defaults in effect throughout your CloudStack
deployment.

#. 

   Log in to the UI as administrator.

#. 

   In the left navigation bar, click Global Settings.

#. 

   In Select View, choose one of the following:

   -  

      Global Settings. This displays a list of the parameters with brief
      descriptions and current values.

   -  

      Hypervisor Capabilities. This displays a list of hypervisor
      versions with the maximum number of guests supported for each.

#. 

   Use the search box to narrow down the list to those you are
   interested in.

#. 

   In the Actions column, click the Edit icon to modify a value. If you
   are viewing Hypervisor Capabilities, you must click the name of the
   hypervisor first to display the editing screen.

Setting Local Configuration Parameters
--------------------------------------

Use the following steps to set local configuration parameters for an
account, zone, cluster, or primary storage. These values will override
the global configuration settings.

#. 

   Log in to the UI as administrator.

#. 

   In the left navigation bar, click Infrastructure or Accounts,
   depending on where you want to set a value.

#. 

   Find the name of the particular resource that you want to work with.
   For example, if you are in Infrastructure, click View All on the
   Zones, Clusters, or Primary Storage area.

#. 

   Click the name of the resource where you want to set a limit.

#. 

   Click the Settings tab.

#. 

   Use the search box to narrow down the list to those you are
   interested in.

#. 

   In the Actions column, click the Edit icon to modify a value.

Granular Global Configuration Parameters
----------------------------------------

The following global configuration parameters have been made more
granular. The parameters are listed under three different scopes:
account, cluster, and zone.

Field

Field

Value

account

remote.access.vpn.client.iprange

The range of IPs to be allocated to remotely access the VPN clients. The
first IP in the range is used by the VPN server.

account

allow.public.user.templates

If false, users will not be able to create public templates.

account

use.system.public.ips

If true and if an account has one or more dedicated public IP ranges,
IPs are acquired from the system pool after all the IPs dedicated to the
account have been consumed.

account

use.system.guest.vlans

If true and if an account has one or more dedicated guest VLAN ranges,
VLANs are allocated from the system pool after all the VLANs dedicated
to the account have been consumed.

cluster

cluster.storage.allocated.capacity.notificationthreshold

The percentage, as a value between 0 and 1, of allocated storage
utilization above which alerts are sent that the storage is below the
threshold.

cluster

cluster.storage.capacity.notificationthreshold

The percentage, as a value between 0 and 1, of storage utilization above
which alerts are sent that the available storage is below the threshold.

cluster

cluster.cpu.allocated.capacity.notificationthreshold

The percentage, as a value between 0 and 1, of cpu utilization above
which alerts are sent that the available CPU is below the threshold.

cluster

cluster.memory.allocated.capacity.notificationthreshold

The percentage, as a value between 0 and 1, of memory utilization above
which alerts are sent that the available memory is below the threshold.

cluster

cluster.cpu.allocated.capacity.disablethreshold

The percentage, as a value between 0 and 1, of CPU utilization above
which allocators will disable that cluster from further usage. Keep the
corresponding notification threshold lower than this value to be
notified beforehand.

cluster

cluster.memory.allocated.capacity.disablethreshold

The percentage, as a value between 0 and 1, of memory utilization above
which allocators will disable that cluster from further usage. Keep the
corresponding notification threshold lower than this value to be
notified beforehand.

cluster

cpu.overprovisioning.factor

Used for CPU over-provisioning calculation; the available CPU will be
the mathematical product of actualCpuCapacity and
cpu.overprovisioning.factor.

cluster

mem.overprovisioning.factor

Used for memory over-provisioning calculation.

cluster

vmware.reserve.cpu

Specify whether or not to reserve CPU when not over-provisioning; In
case of CPU over-provisioning, CPU is always reserved.

cluster

vmware.reserve.mem

Specify whether or not to reserve memory when not over-provisioning; In
case of memory over-provisioning memory is always reserved.

zone

pool.storage.allocated.capacity.disablethreshold

The percentage, as a value between 0 and 1, of allocated storage
utilization above which allocators will disable that pool because the
available allocated storage is below the threshold.

zone

pool.storage.capacity.disablethreshold

The percentage, as a value between 0 and 1, of storage utilization above
which allocators will disable the pool because the available storage
capacity is below the threshold.

zone

storage.overprovisioning.factor

Used for storage over-provisioning calculation; available storage will
be the mathematical product of actualStorageSize and
storage.overprovisioning.factor.

zone

network.throttling.rate

Default data transfer rate in megabits per second allowed in a network.

zone

guest.domain.suffix

Default domain name for VMs inside a virtual networks with a router.

zone

router.template.xen

Name of the default router template on Xenserver.

zone

router.template.kvm

Name of the default router template on KVM.

zone

router.template.vmware

Name of the default router template on VMware.

zone

enable.dynamic.scale.vm

Enable or diable dynamically scaling of a VM.

zone

use.external.dns

Bypass internal DNS, and use the external DNS1 and DNS2

zone

blacklisted.routes

Routes that are blacklisted cannot be used for creating static routes
for a VPC Private Gateway.



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
