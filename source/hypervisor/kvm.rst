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


KVM Hypervisor Host Installation
--------------------------------

System Requirements for KVM Hypervisor Hosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

KVM is included with a variety of Linux-based operating systems.
Although you are not required to run these distributions, the following
are recommended:

-  CentOS / RHEL: 6.3

-  Ubuntu: 12.04(.1)

The main requirement for KVM hypervisors is the libvirt and Qemu
version. No matter what Linux distribution you are using, make sure the
following requirements are met:

-  libvirt: 0.9.4 or higher

-  Qemu/KVM: 1.0 or higher

The default bridge in CloudStack is the Linux native bridge
implementation (bridge module). CloudStack includes an option to work
with OpenVswitch, the requirements are listed below

-  libvirt: 0.9.11 or higher

-  openvswitch: 1.7.1 or higher

In addition, the following hardware requirements apply:

-  Within a single cluster, the hosts must be of the same distribution
   version.

-  All hosts within a cluster must be homogenous. The CPUs must be of
   the same type, count, and feature flags.

-  Must support HVM (Intel-VT or AMD-V enabled)

-  64-bit x86 CPU (more cores results in better performance)

-  4 GB of memory

-  At least 1 NIC

-  When you deploy CloudStack, the hypervisor host must not have any VMs
   already running


KVM Installation Overview
~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to use the Linux Kernel Virtual Machine (KVM) hypervisor to
run guest virtual machines, install KVM on the host(s) in your cloud.
The material in this section doesn't duplicate KVM installation docs. It
provides the CloudStack-specific steps that are needed to prepare a KVM
host to work with CloudStack.

.. warning::
   Before continuing, make sure that you have applied the latest updates to 
   your host.

.. warning::
   It is NOT recommended to run services on this host not controlled by 
   CloudStack.

The procedure for installing a KVM Hypervisor Host is:

#. Prepare the Operating System

#. Install and configure libvirt

#. Configure Security Policies (AppArmor and SELinux)

#. Install and configure the Agent


Prepare the Operating System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OS of the Host must be prepared to host the CloudStack Agent and run
KVM instances.

#. Log in to your OS as root.

#. Check for a fully qualified hostname.

   .. sourcecode:: bash

      $ hostname --fqdn

   This should return a fully qualified hostname such as
   "kvm1.lab.example.org". If it does not, edit /etc/hosts so that it
   does.

#. Make sure that the machine can reach the Internet.

   .. sourcecode:: bash

      $ ping www.cloudstack.org

#. Turn on NTP for time synchronization.

   .. note::
      NTP is required to synchronize the clocks of the servers in your
      cloud. Unsynchronized clocks can cause unexpected problems.

   #. Install NTP

      .. sourcecode:: bash

         $ yum install ntp

      .. sourcecode:: bash

         $ apt-get install openntpd

#. Repeat all of these steps on every hypervisor host.


Install and configure the Agent
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To manage KVM instances on the host CloudStack uses a Agent. This Agent
communicates with the Management server and controls all the instances
on the host.

First we start by installing the agent:

In RHEL or CentOS:

.. sourcecode:: bash

   $ yum install cloudstack-agent

In Ubuntu:

.. sourcecode:: bash

   $ apt-get install cloudstack-agent

The host is now ready to be added to a cluster. This is covered in a
later section, see :ref:`adding-a-host`. It is
recommended that you continue to read the documentation before adding
the host!


Configure CPU model for KVM guest (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In additional,the CloudStack Agent allows host administrator to control
the guest CPU model which is exposed to KVM instances. By default, the
CPU model of KVM instance is likely QEMU Virtual CPU version x.x.x with
least CPU features exposed. There are a couple of reasons to specify the
CPU model:

-  To maximise performance of instances by exposing new host CPU
   features to the KVM instances;

-  To ensure a consistent default CPU across all machines,removing
   reliance of variable QEMU defaults;

For the most part it will be sufficient for the host administrator to
specify the guest CPU config in the per-host configuration file
(/etc/cloudstack/agent/agent.properties). This will be achieved by
introducing two new configuration parameters:

.. sourcecode:: bash

   guest.cpu.mode=custom|host-model|host-passthrough
   guest.cpu.model=from /usr/share/libvirt/cpu_map.xml(only valid when guest.cpu.mode=custom)

There are three choices to fulfill the cpu model changes:

#. **custom:** you can explicitly specify one of the supported named
   model in /usr/share/libvirt/cpu\_map.xml

#. **host-model:** libvirt will identify the CPU model in
   /usr/share/libvirt/cpu\_map.xml which most closely matches the host,
   and then request additional CPU flags to complete the match. This
   should give close to maximum functionality/performance, which
   maintaining good reliability/compatibility if the guest is migrated
   to another host with slightly different host CPUs.

#. **host-passthrough:** libvirt will tell KVM to passthrough the host
   CPU with no modifications. The difference to host-model, instead of
   just matching feature flags, every last detail of the host CPU is
   matched. This gives absolutely best performance, and can be important
   to some apps which check low level CPU details, but it comes at a
   cost with respect to migration: the guest can only be migrated to an
   exactly matching host CPU.

Here are some examples:

-  custom

   .. sourcecode:: bash

      guest.cpu.mode=custom
      guest.cpu.model=SandyBridge

-  host-model

   .. sourcecode:: bash

      guest.cpu.mode=host-model

-  host-passthrough

   .. sourcecode:: bash

      guest.cpu.mode=host-passthrough

.. note:: 
   host-passthrough may lead to migration failure,if you have this problem, 
   you should use host-model or custom


Install and Configure libvirt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack uses libvirt for managing virtual machines. Therefore it is
vital that libvirt is configured correctly. Libvirt is a dependency of
cloudstack-agent and should already be installed.

#. In order to have live migration working libvirt has to listen for
   unsecured TCP connections. We also need to turn off libvirts attempt
   to use Multicast DNS advertising. Both of these settings are in
   ``/etc/libvirt/libvirtd.conf``

   Set the following parameters:

   .. sourcecode:: bash

      listen_tls = 0

   .. sourcecode:: bash

      listen_tcp = 1

   .. sourcecode:: bash

      tcp_port = "16509"

   .. sourcecode:: bash

      auth_tcp = "none"

   .. sourcecode:: bash

      mdns_adv = 0

#. Turning on "listen\_tcp" in libvirtd.conf is not enough, we have to
   change the parameters as well:

   On RHEL or CentOS modify ``/etc/sysconfig/libvirtd``:

   Uncomment the following line:

   .. sourcecode:: bash

      #LIBVIRTD_ARGS="--listen"

   On Ubuntu: modify ``/etc/default/libvirt-bin``

   Add "-l" to the following line

   .. sourcecode:: bash

      libvirtd_opts="-d"

   so it looks like:

   .. sourcecode:: bash

      libvirtd_opts="-d -l"

#. Restart libvirt

   In RHEL or CentOS:

   .. sourcecode:: bash

      $ service libvirtd restart

   In Ubuntu:

   .. sourcecode:: bash

      $ service libvirt-bin restart


Configure the Security Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack does various things which can be blocked by security
mechanisms like AppArmor and SELinux. These have to be disabled to
ensure the Agent has all the required permissions.

#. Configure SELinux (RHEL and CentOS)

   #. Check to see whether SELinux is installed on your machine. If not,
      you can skip this section.

      In RHEL or CentOS, SELinux is installed and enabled by default.
      You can verify this with:

      .. sourcecode:: bash

         $ rpm -qa | grep selinux

   #. Set the SELINUX variable in ``/etc/selinux/config`` to
      "permissive". This ensures that the permissive setting will be
      maintained after a system reboot.

      In RHEL or CentOS:

      .. sourcecode:: bash

         $ vi /etc/selinux/config

      Change the following line

      .. sourcecode:: bash

         SELINUX=enforcing

      to this

      .. sourcecode:: bash

         SELINUX=permissive

   #. Then set SELinux to permissive starting immediately, without
      requiring a system reboot.

      .. sourcecode:: bash

         $ setenforce permissive

#. Configure Apparmor (Ubuntu)

   #. Check to see whether AppArmor is installed on your machine. If
      not, you can skip this section.

      In Ubuntu AppArmor is installed and enabled by default. You can
      verify this with:

      .. sourcecode:: bash

         $ dpkg --list 'apparmor'

   #. Disable the AppArmor profiles for libvirt

      .. sourcecode:: bash

         $ ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/

      .. sourcecode:: bash

         $ ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/

      .. sourcecode:: bash

         $ apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd

      .. sourcecode:: bash

         $ apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper


Configure the network bridges
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning:: 
   This is a very important section, please make sure you read this thoroughly.

.. note:: 
   This section details how to configure bridges using the native 
   implementation in Linux. Please refer to the next section if you intend to 
   use OpenVswitch

In order to forward traffic to your instances you will need at least two
bridges: *public* and *private*.

By default these bridges are called *cloudbr0* and *cloudbr1*, but you
do have to make sure they are available on each hypervisor.

The most important factor is that you keep the configuration consistent
on all your hypervisors.


Network example
^^^^^^^^^^^^^^^

There are many ways to configure your network. In the Basic networking
mode you should have two (V)LAN's, one for your private network and one
for the public network.

We assume that the hypervisor has one NIC (eth0) with three tagged
VLAN's:

#. VLAN 100 for management of the hypervisor

#. VLAN 200 for public network of the instances (cloudbr0)

#. VLAN 300 for private network of the instances (cloudbr1)

On VLAN 100 we give the Hypervisor the IP-Address 192.168.42.11/24 with
the gateway 192.168.42.1

.. note:: 
   The Hypervisor and Management server don't have to be in the same subnet!


Configuring the network bridges
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It depends on the distribution you are using how to configure these,
below you'll find examples for RHEL/CentOS and Ubuntu.

.. note:: 
   The goal is to have two bridges called 'cloudbr0' and 'cloudbr1' after this 
   section. This should be used as a guideline only. The exact configuration 
   will depend on your network layout.


Configure in RHEL or CentOS
'''''''''''''''''''''''''''

The required packages were installed when libvirt was installed, we can
proceed to configuring the network.

First we configure eth0

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-eth0

Make sure it looks similar to:

.. sourcecode:: bash

   DEVICE=eth0
   HWADDR=00:04:xx:xx:xx:xx
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   TYPE=Ethernet

We now have to configure the three VLAN interfaces:

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-eth0.100

.. sourcecode:: bash

   DEVICE=eth0.100
   HWADDR=00:04:xx:xx:xx:xx
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   TYPE=Ethernet
   VLAN=yes
   IPADDR=192.168.42.11
   GATEWAY=192.168.42.1
   NETMASK=255.255.255.0

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-eth0.200

.. sourcecode:: bash

   DEVICE=eth0.200
   HWADDR=00:04:xx:xx:xx:xx
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   TYPE=Ethernet
   VLAN=yes
   BRIDGE=cloudbr0

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-eth0.300

.. sourcecode:: bash

   DEVICE=eth0.300
   HWADDR=00:04:xx:xx:xx:xx
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   TYPE=Ethernet
   VLAN=yes
   BRIDGE=cloudbr1

Now we have the VLAN interfaces configured we can add the bridges on top
of them.

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-cloudbr0

Now we just configure it is a plain bridge without an IP-Address

.. sourcecode:: bash

   DEVICE=cloudbr0
   TYPE=Bridge
   ONBOOT=yes
   BOOTPROTO=none
   IPV6INIT=no
   IPV6_AUTOCONF=no
   DELAY=5
   STP=yes

We do the same for cloudbr1

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-cloudbr1

.. sourcecode:: bash

   DEVICE=cloudbr1
   TYPE=Bridge
   ONBOOT=yes
   BOOTPROTO=none
   IPV6INIT=no
   IPV6_AUTOCONF=no
   DELAY=5
   STP=yes

With this configuration you should be able to restart the network,
although a reboot is recommended to see if everything works properly.

.. warning:: 
   Make sure you have an alternative way like IPMI or ILO to reach the machine 
   in case you made a configuration error and the network stops functioning!


Configure in Ubuntu
'''''''''''''''''''

All the required packages were installed when you installed libvirt, so
we only have to configure the network.

.. sourcecode:: bash

   $ vi /etc/network/interfaces

Modify the interfaces file to look like this:

.. sourcecode:: bash

   auto lo
   iface lo inet loopback

   # The primary network interface
   auto eth0.100
   iface eth0.100 inet static
       address 192.168.42.11
       netmask 255.255.255.240
       gateway 192.168.42.1
       dns-nameservers 8.8.8.8 8.8.4.4
       dns-domain lab.example.org

   # Public network
   auto cloudbr0
   iface cloudbr0 inet manual
       bridge_ports eth0.200
       bridge_fd 5
       bridge_stp off
       bridge_maxwait 1

   # Private network
   auto cloudbr1
   iface cloudbr1 inet manual
       bridge_ports eth0.300
       bridge_fd 5
       bridge_stp off
       bridge_maxwait 1

With this configuration you should be able to restart the network,
although a reboot is recommended to see if everything works properly.

.. warning:: 
   Make sure you have an alternative way like IPMI or ILO to reach the machine 
   in case you made a configuration error and the network stops functioning!


Configure the network using OpenVswitch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning::
   This is a very important section, please make sure you read this thoroughly.

In order to forward traffic to your instances you will need at least two
bridges: *public* and *private*.

By default these bridges are called *cloudbr0* and *cloudbr1*, but you
do have to make sure they are available on each hypervisor.

The most important factor is that you keep the configuration consistent
on all your hypervisors.


Preparing
^^^^^^^^^

To make sure that the native bridge module will not interfere with
openvswitch the bridge module should be added to the blacklist. See the
modprobe documentation for your distribution on where to find the
blacklist. Make sure the module is not loaded either by rebooting or
executing rmmod bridge before executing next steps.

The network configurations below depend on the ifup-ovs and ifdown-ovs
scripts which are part of the openvswitch installation. They should be
installed in /etc/sysconfig/network-scripts/


Network example
^^^^^^^^^^^^^^^

There are many ways to configure your network. In the Basic networking
mode you should have two (V)LAN's, one for your private network and one
for the public network.

We assume that the hypervisor has one NIC (eth0) with three tagged
VLAN's:

#. VLAN 100 for management of the hypervisor

#. VLAN 200 for public network of the instances (cloudbr0)

#. VLAN 300 for private network of the instances (cloudbr1)

On VLAN 100 we give the Hypervisor the IP-Address 192.168.42.11/24 with
the gateway 192.168.42.1

.. note::
   The Hypervisor and Management server don't have to be in the same subnet!


Configuring the network bridges
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It depends on the distribution you are using how to configure these,
below you'll find examples for RHEL/CentOS.

.. note:: 
   The goal is to have three bridges called 'mgmt0', 'cloudbr0' and 'cloudbr1' 
   after this section. This should be used as a guideline only. The exact 
   configuration will depend on your network layout.


Configure OpenVswitch
'''''''''''''''''''''

The network interfaces using OpenVswitch are created using the ovs-vsctl
command. This command will configure the interfaces and persist them to
the OpenVswitch database.

First we create a main bridge connected to the eth0 interface. Next we
create three fake bridges, each connected to a specific vlan tag.

.. sourcecode:: bash

   # ovs-vsctl add-br cloudbr
   # ovs-vsctl add-port cloudbr eth0 
   # ovs-vsctl set port cloudbr trunks=100,200,300
   # ovs-vsctl add-br mgmt0 cloudbr 100
   # ovs-vsctl add-br cloudbr0 cloudbr 200
   # ovs-vsctl add-br cloudbr1 cloudbr 300


Configure in RHEL or CentOS
'''''''''''''''''''''''''''

The required packages were installed when openvswitch and libvirt were
installed, we can proceed to configuring the network.

First we configure eth0

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-eth0

Make sure it looks similar to:

.. sourcecode:: bash

   DEVICE=eth0
   HWADDR=00:04:xx:xx:xx:xx
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   TYPE=Ethernet

We have to configure the base bridge with the trunk.

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-cloudbr

.. sourcecode:: bash

   DEVICE=cloudbr
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   DEVICETYPE=ovs
   TYPE=OVSBridge

We now have to configure the three VLAN bridges:

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-mgmt0

.. sourcecode:: bash

   DEVICE=mgmt0
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=static
   DEVICETYPE=ovs
   TYPE=OVSBridge
   IPADDR=192.168.42.11
   GATEWAY=192.168.42.1
   NETMASK=255.255.255.0

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-cloudbr0

.. sourcecode:: bash

   DEVICE=cloudbr0
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   DEVICETYPE=ovs
   TYPE=OVSBridge

.. sourcecode:: bash

   $ vi /etc/sysconfig/network-scripts/ifcfg-cloudbr1

.. sourcecode:: bash

   DEVICE=cloudbr1
   ONBOOT=yes
   HOTPLUG=no
   BOOTPROTO=none
   TYPE=OVSBridge
   DEVICETYPE=ovs

With this configuration you should be able to restart the network,
although a reboot is recommended to see if everything works properly.

.. warning:: 
   Make sure you have an alternative way like IPMI or ILO to reach the machine 
   in case you made a configuration error and the network stops functioning!


Configuring the firewall
~~~~~~~~~~~~~~~~~~~~~~~~

The hypervisor needs to be able to communicate with other hypervisors
and the management server needs to be able to reach the hypervisor.

In order to do so we have to open the following TCP ports (if you are
using a firewall):

#. 22 (SSH)

#. 1798

#. 16509 (libvirt)

#. 5900 - 6100 (VNC consoles)

#. 49152 - 49216 (libvirt live migration)

It depends on the firewall you are using how to open these ports. Below
you'll find examples how to open these ports in RHEL/CentOS and Ubuntu.


Open ports in RHEL/CentOS
^^^^^^^^^^^^^^^^^^^^^^^^^

RHEL and CentOS use iptables for firewalling the system, you can open
extra ports by executing the following iptable commands:

.. sourcecode:: bash

   $ iptables -I INPUT -p tcp -m tcp --dport 22 -j ACCEPT

.. sourcecode:: bash

   $ iptables -I INPUT -p tcp -m tcp --dport 1798 -j ACCEPT

.. sourcecode:: bash

   $ iptables -I INPUT -p tcp -m tcp --dport 16509 -j ACCEPT

.. sourcecode:: bash

   $ iptables -I INPUT -p tcp -m tcp --dport 5900:6100 -j ACCEPT

.. sourcecode:: bash

   $ iptables -I INPUT -p tcp -m tcp --dport 49152:49216 -j ACCEPT

These iptable settings are not persistent accross reboots, we have to
save them first.

.. sourcecode:: bash

   $ iptables-save > /etc/sysconfig/iptables


Open ports in Ubuntu
^^^^^^^^^^^^^^^^^^^^

The default firewall under Ubuntu is UFW (Uncomplicated FireWall), which
is a Python wrapper around iptables.

To open the required ports, execute the following commands:

.. sourcecode:: bash

   $ ufw allow proto tcp from any to any port 22

.. sourcecode:: bash

   $ ufw allow proto tcp from any to any port 1798

.. sourcecode:: bash

   $ ufw allow proto tcp from any to any port 16509

.. sourcecode:: bash

   $ ufw allow proto tcp from any to any port 5900:6100

.. sourcecode:: bash

   $ ufw allow proto tcp from any to any port 49152:49216

.. note:: 
   By default UFW is not enabled on Ubuntu. Executing these commands with the 
   firewall disabled does not enable the firewall.


Add the host to CloudStack
~~~~~~~~~~~~~~~~~~~~~~~~~~

The host is now ready to be added to a cluster. This is covered in a
later section, see :ref:`adding-a-host`. It is
recommended that you continue to read the documentation before adding
the host!


Hypervisor Support for Primary Storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table shows storage options and parameters for different
hypervisors.

==================================================  =============  ====================  ==========================  =======
Primary Storage Type                                vSphere        XenServer             KVM                         Hyper-V
==================================================  =============  ====================  ==========================  =======
****Format for Disks, Templates, and Snapshots****  VMDK           VHD                   QCOW2                       VHD
**iSCSI support**                                   VMFS           CLVM                  Yes, via Shared Mountpoint  No
**Fiber Channel support**                           VMFS           Yes, via Existing SR  Yes, via Shared Mountpoint  No
**NFS support**                                     Yes            Yes                   Yes                         No
**Local storage support**                           Yes            Yes                   Yes                         Yes
**Storage over-provisioning**                       NFS and iSCSI  NFS                   NFS                         No
**SMB/CIFS**                                        No             No                    No                          Yes
==================================================  =============  ====================  ==========================  =======

XenServer uses a clustered LVM system to store VM images on iSCSI and
Fiber Channel volumes and does not support over-provisioning in the
hypervisor. The storage server itself, however, can support
thin-provisioning. As a result the CloudStack can still support storage
over-provisioning by running on thin-provisioned storage volumes.

KVM supports "Shared Mountpoint" storage. A shared mountpoint is a file
system path local to each server in a given cluster. The path must be
the same across all Hosts in the cluster, for example /mnt/primary1.
This shared mountpoint is assumed to be a clustered filesystem such as
OCFS2. In this case the CloudStack does not attempt to mount or unmount
the storage as is done with NFS. The CloudStack requires that the
administrator insure that the storage is available

With NFS storage, CloudStack manages the overprovisioning. In this case
the global configuration parameter storage.overprovisioning.factor
controls the degree of overprovisioning. This is independent of
hypervisor type.

Local storage is an option for primary storage for vSphere, XenServer,
and KVM. When the local disk option is enabled, a local disk storage
pool is automatically created on each host. To use local storage for the
System Virtual Machines (such as the Virtual Router), set
system.vm.use.local.storage to true in global configuration.

CloudStack supports multiple primary storage pools in a Cluster. For
example, you could provision 2 NFS servers in primary storage. Or you
could provision 1 iSCSI LUN initially and then add a second iSCSI LUN
when the first approaches capacity.
