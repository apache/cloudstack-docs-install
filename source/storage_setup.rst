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

Storage Setup
=============

CloudStack is designed to work with a wide variety of commodity and
enterprise-grade storage. Local disk may be used as well, if supported
by the selected hypervisor. Storage type support for guest virtual disks
differs based on hypervisor selection.

XenServer

vSphere

KVM

NFS

Supported

Supported

Supported

iSCSI

Supported

Supported via VMFS

Supported via Clustered Filesystems

Fiber Channel

Supported via Pre-existing SR

Supported

Supported via Clustered Filesystems

Local Disk

Supported

Supported

Supported

The use of the Cluster Logical Volume Manager (CLVM) for KVM is not
officially supported with CloudStack.

Small-Scale Setup
-----------------

In a small-scale setup, a single NFS server can function as both primary
and secondary storage. The NFS server just needs to export two separate
shares, one for primary storage and the other for secondary storage.

Secondary Storage
-----------------

CloudStack is designed to work with any scalable secondary storage
system. The only requirement is the secondary storage system supports
the NFS protocol.

.. note:: The storage server should be a machine with a large number of disks. The
disks should ideally be managed by a hardware RAID controller. Modern
hardware RAID controllers support hot plug functionality independent of
the operating system so you can replace faulty disks without impacting
the running operating system.

Example Configurations
----------------------

In this section we go through a few examples of how to set up storage to
work properly on a few types of NFS and iSCSI storage systems.

Linux NFS on Local Disks and DAS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section describes how to configure an NFS export on a standard
Linux installation. The exact commands might vary depending on the
operating system version.

#. 

   Install the RHEL/CentOS distribution on the storage server.

#. 

   If the root volume is more than 2 TB in size, create a smaller boot
   volume to install RHEL/CentOS. A root volume of 20 GB should be
   sufficient.

#. 

   After the system is installed, create a directory called /export.
   This can each be a directory in the root partition itself or a mount
   point for a large disk volume.

#. 

   If you have more than 16TB of storage on one host, create multiple
   EXT3 file systems and multiple NFS exports. Individual EXT3 file
   systems cannot exceed 16TB.

#. 

   After /export directory is created, run the following command to
   configure it as an NFS export.

   .. code:: bash

       # echo "/export <CIDR>(rw,async,no_root_squash,no_subtree_check)" >> /etc/exports

   Adjust the above command to suit your deployment needs.

   -  

      **Limiting NFS export.** It is highly recommended that you limit
      the NFS export to a particular subnet by specifying a subnet mask
      (e.g.,”192.168.1.0/24”). By allowing access from only within the
      expected cluster, you avoid having non-pool member mount the
      storage. The limit you place must include the management
      network(s) and the storage network(s). If the two are the same
      network then one CIDR is sufficient. If you have a separate
      storage network you must provide separate CIDR’s for both or one
      CIDR that is broad enough to span both.

      The following is an example with separate CIDRs:

      .. code:: bash

          /export 192.168.1.0/24(rw,async,no_root_squash,no_subtree_check) 10.50.1.0/24(rw,async,no_root_squash,no_subtree_check)

   -  

      **Removing the async flag.** The async flag improves performance
      by allowing the NFS server to respond before writes are committed
      to the disk. Remove the async flag in your mission critical
      production deployment.

#. 

   Run the following command to enable NFS service.

   .. code:: bash

       # chkconfig nfs on

#. 

   Edit the /etc/sysconfig/nfs file and uncomment the following lines.

   .. code:: bash

       LOCKD_TCPPORT=32803
       LOCKD_UDPPORT=32769
       MOUNTD_PORT=892
       RQUOTAD_PORT=875
       STATD_PORT=662
       STATD_OUTGOING_PORT=2020

#. 

   Edit the /etc/sysconfig/iptables file and add the following lines at
   the beginning of the INPUT chain.

   .. code:: bash

       -A INPUT -m state --state NEW -p udp --dport 111 -j ACCEPT
       -A INPUT -m state --state NEW -p tcp --dport 111 -j ACCEPT
       -A INPUT -m state --state NEW -p tcp --dport 2049 -j ACCEPT
       -A INPUT -m state --state NEW -p tcp --dport 32803 -j ACCEPT
       -A INPUT -m state --state NEW -p udp --dport 32769 -j ACCEPT
       -A INPUT -m state --state NEW -p tcp --dport 892 -j ACCEPT
       -A INPUT -m state --state NEW -p udp --dport 892 -j ACCEPT
       -A INPUT -m state --state NEW -p tcp --dport 875 -j ACCEPT
       -A INPUT -m state --state NEW -p udp --dport 875 -j ACCEPT
       -A INPUT -m state --state NEW -p tcp --dport 662 -j ACCEPT
       -A INPUT -m state --state NEW -p udp --dport 662 -j ACCEPT

#. 

   Reboot the server.

   An NFS share called /export is now set up.

.. note:: When copying and pasting a command, be sure the command has pasted as a
single line before executing. Some document viewers may introduce
unwanted line breaks in copied text.

Linux NFS on iSCSI
~~~~~~~~~~~~~~~~~~

Use the following steps to set up a Linux NFS server export on an iSCSI
volume. These steps apply to RHEL/CentOS 5 distributions.

#. 

   Install iscsiadm.

   .. code:: bash

       # yum install iscsi-initiator-utils
       # service iscsi start
       # chkconfig --add iscsi
       # chkconfig iscsi on

#. 

   Discover the iSCSI target.

   .. code:: bash

       # iscsiadm -m discovery -t st -p <iSCSI Server IP address>:3260

   For example:

   .. code:: bash

       # iscsiadm -m discovery -t st -p 172.23.10.240:3260
                 172.23.10.240:3260,1 iqn.2001-05.com.equallogic:0-8a0906-83bcb3401-16e0002fd0a46f3d-rhel5-test

#. 

   Log in.

   .. code:: bash

       # iscsiadm -m node -T <Complete Target Name> -l -p <Group IP>:3260

   For example:

   .. code:: bash

       # iscsiadm -m node -l -T iqn.2001-05.com.equallogic:83bcb3401-16e0002fd0a46f3d-rhel5-test -p 172.23.10.240:3260

#. 

   Discover the SCSI disk. For example:

   .. code:: bash

       # iscsiadm -m session -P3 | grep Attached
       Attached scsi disk sdb State: running

#. 

   Format the disk as ext3 and mount the volume.

   .. code:: bash

       # mkfs.ext3 /dev/sdb
       # mkdir -p /export
       # mount /dev/sdb /export

#. 

   Add the disk to /etc/fstab to make sure it gets mounted on boot.

   .. code:: bash

       /dev/sdb /export ext3 _netdev 0 0

Now you can set up /export as an NFS share.

-  

   **Limiting NFS export.** In order to avoid data loss, it is highly
   recommended that you limit the NFS export to a particular subnet by
   specifying a subnet mask (e.g.,”192.168.1.0/24”). By allowing access
   from only within the expected cluster, you avoid having non-pool
   member mount the storage and inadvertently delete all its data. The
   limit you place must include the management network(s) and the
   storage network(s). If the two are the same network then one CIDR is
   sufficient. If you have a separate storage network you must provide
   separate CIDRs for both or one CIDR that is broad enough to span
   both.

   The following is an example with separate CIDRs:

   .. code:: bash

       /export 192.168.1.0/24(rw,async,no_root_squash,no_subtree_check) 10.50.1.0/24(rw,async,no_root_squash,no_subtree_check)

-  

   **Removing the async flag.** The async flag improves performance by
   allowing the NFS server to respond before writes are committed to the
   disk. Remove the async flag in your mission critical production
   deployment.


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
