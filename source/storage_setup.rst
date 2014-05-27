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

=============  ==============================  ==================  ===================================
Storage Type   XenServer                       vSphere             KVM
=============  ==============================  ==================  ===================================
NFS            Supported                       Supported           Supported
iSCSI          Supported                       Supported via VMFS  Supported via Clustered Filesystems
Fiber Channel  Supported via Pre-existing SR   Supported           Supported via Clustered Filesystems
Local Disk     Supported                       Supported           Supported
=============  ==============================  ==================  ===================================

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

.. note::
   The storage server should be a machine with a large number of disks. The 
   disks should ideally be managed by a hardware RAID controller. Modern 
   hardware RAID controllers support hot plug functionality independent of the 
   operating system so you can replace faulty disks without impacting the 
   running operating system.


Example Configurations
----------------------

In this section we go through a few examples of how to set up storage to
work properly on a few types of NFS and iSCSI storage systems.


Linux NFS on Local Disks and DAS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section describes how to configure an NFS export on a standard
Linux installation. The exact commands might vary depending on the
operating system version.

#. Install the RHEL/CentOS distribution on the storage server.

#. If the root volume is more than 2 TB in size, create a smaller boot
   volume to install RHEL/CentOS. A root volume of 20 GB should be
   sufficient.

#. After the system is installed, create a directory called /export.
   This can each be a directory in the root partition itself or a mount
   point for a large disk volume.

#. If you have more than 16TB of storage on one host, create multiple
   EXT3 file systems and multiple NFS exports. Individual EXT3 file
   systems cannot exceed 16TB.

#. After /export directory is created, run the following command to
   configure it as an NFS export.

   .. sourcecode:: bash

      # echo "/export <CIDR>(rw,async,no_root_squash,no_subtree_check)" >> /etc/exports

   Adjust the above command to suit your deployment needs.

   -  **Limiting NFS export.** It is highly recommended that you limit
      the NFS export to a particular subnet by specifying a subnet mask
      (e.g.,”192.168.1.0/24”). By allowing access from only within the
      expected cluster, you avoid having non-pool member mount the
      storage. The limit you place must include the management
      network(s) and the storage network(s). If the two are the same
      network then one CIDR is sufficient. If you have a separate
      storage network you must provide separate CIDR’s for both or one
      CIDR that is broad enough to span both.

      The following is an example with separate CIDRs:

      .. sourcecode:: bash

         /export 192.168.1.0/24(rw,async,no_root_squash,no_subtree_check) 10.50.1.0/24(rw,async,no_root_squash,no_subtree_check)

   -  **Removing the async flag.** The async flag improves performance
      by allowing the NFS server to respond before writes are committed
      to the disk. Remove the async flag in your mission critical
      production deployment.

#. Run the following command to enable NFS service.

   .. sourcecode:: bash

      # chkconfig nfs on

#. Edit the /etc/sysconfig/nfs file and uncomment the following lines.

   .. sourcecode:: bash

      LOCKD_TCPPORT=32803
      LOCKD_UDPPORT=32769
      MOUNTD_PORT=892
      RQUOTAD_PORT=875
      STATD_PORT=662
      STATD_OUTGOING_PORT=2020

#. Edit the /etc/sysconfig/iptables file and add the following lines at
   the beginning of the INPUT chain.

   .. sourcecode:: bash

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

#. Reboot the server.

   An NFS share called /export is now set up.

.. note::
   When copying and pasting a command, be sure the command has pasted as a 
   single line before executing. Some document viewers may introduce unwanted 
   line breaks in copied text.


Linux NFS on iSCSI
~~~~~~~~~~~~~~~~~~

Use the following steps to set up a Linux NFS server export on an iSCSI
volume. These steps apply to RHEL/CentOS 5 distributions.

#. Install iscsiadm.

   .. sourcecode:: bash

      # yum install iscsi-initiator-utils
      # service iscsi start
      # chkconfig --add iscsi
      # chkconfig iscsi on

#. Discover the iSCSI target.

   .. sourcecode:: bash

      # iscsiadm -m discovery -t st -p <iSCSI Server IP address>:3260

   For example:

   .. sourcecode:: bash

      # iscsiadm -m discovery -t st -p 172.23.10.240:3260 172.23.10.240:3260,1 iqn.2001-05.com.equallogic:0-8a0906-83bcb3401-16e0002fd0a46f3d-rhel5-test

#. Log in.

   .. sourcecode:: bash

      # iscsiadm -m node -T <Complete Target Name> -l -p <Group IP>:3260

   For example:

   .. sourcecode:: bash

      # iscsiadm -m node -l -T iqn.2001-05.com.equallogic:83bcb3401-16e0002fd0a46f3d-rhel5-test -p 172.23.10.240:3260

#. Discover the SCSI disk. For example:

   .. sourcecode:: bash

      # iscsiadm -m session -P3 | grep Attached
      Attached scsi disk sdb State: running

#. Format the disk as ext3 and mount the volume.

   .. sourcecode:: bash

      # mkfs.ext3 /dev/sdb
      # mkdir -p /export
      # mount /dev/sdb /export

#. Add the disk to /etc/fstab to make sure it gets mounted on boot.

   .. sourcecode:: bash

      /dev/sdb /export ext3 _netdev 0 0

Now you can set up /export as an NFS share.

-  **Limiting NFS export.** In order to avoid data loss, it is highly
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

   .. sourcecode:: bash

      /export 192.168.1.0/24(rw,async,no_root_squash,no_subtree_check) 10.50.1.0/24(rw,async,no_root_squash,no_subtree_check)

-  **Removing the async flag.** The async flag improves performance by
   allowing the NFS server to respond before writes are committed to the
   disk. Remove the async flag in your mission critical production
   deployment.
