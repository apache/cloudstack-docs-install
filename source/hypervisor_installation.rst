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

Hypervisor Installation
=======================

KVM Hypervisor Host Installation
--------------------------------

System Requirements for KVM Hypervisor Hosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

KVM is included with a variety of Linux-based operating systems.
Although you are not required to run these distributions, the following
are recommended:

-  

   CentOS / RHEL: 6.3

-  

   Ubuntu: 12.04(.1)

The main requirement for KVM hypervisors is the libvirt and Qemu
version. No matter what Linux distribution you are using, make sure the
following requirements are met:

-  

   libvirt: 0.9.4 or higher

-  

   Qemu/KVM: 1.0 or higher

The default bridge in CloudStack is the Linux native bridge
implementation (bridge module). CloudStack includes an option to work
with OpenVswitch, the requirements are listed below

-  

   libvirt: 0.9.11 or higher

-  

   openvswitch: 1.7.1 or higher

In addition, the following hardware requirements apply:

-  

   Within a single cluster, the hosts must be of the same distribution
   version.

-  

   All hosts within a cluster must be homogenous. The CPUs must be of
   the same type, count, and feature flags.

-  

   Must support HVM (Intel-VT or AMD-V enabled)

-  

   64-bit x86 CPU (more cores results in better performance)

-  

   4 GB of memory

-  

   At least 1 NIC

-  

   When you deploy CloudStack, the hypervisor host must not have any VMs
   already running

KVM Installation Overview
~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to use the Linux Kernel Virtual Machine (KVM) hypervisor to
run guest virtual machines, install KVM on the host(s) in your cloud.
The material in this section doesn't duplicate KVM installation docs. It
provides the CloudStack-specific steps that are needed to prepare a KVM
host to work with CloudStack.

.. warning:: Before continuing, make sure that you have applied the latest updates to your host.

.. warning:: It is NOT recommended to run services on this host not controlled by CloudStack.

The procedure for installing a KVM Hypervisor Host is:

#. 

   Prepare the Operating System

#. 

   Install and configure libvirt

#. 

   Configure Security Policies (AppArmor and SELinux)

#. 

   Install and configure the Agent

Prepare the Operating System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OS of the Host must be prepared to host the CloudStack Agent and run
KVM instances.

#. 

   Log in to your OS as root.

#. 

   Check for a fully qualified hostname.

   .. code:: bash

       $ hostname --fqdn

   This should return a fully qualified hostname such as
   "kvm1.lab.example.org". If it does not, edit /etc/hosts so that it
   does.

#. 

   Make sure that the machine can reach the Internet.

   .. code:: bash

       $ ping www.cloudstack.org

#. 

   Turn on NTP for time synchronization.

   .. note:: NTP is required to synchronize the clocks of the servers in your
   cloud. Unsynchronized clocks can cause unexpected problems.

   #. 

      Install NTP

      .. code:: bash

          $ yum install ntp

      .. code:: bash

          $ apt-get install openntpd

#. 

   Repeat all of these steps on every hypervisor host.

Install and configure the Agent
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To manage KVM instances on the host CloudStack uses a Agent. This Agent
communicates with the Management server and controls all the instances
on the host.

First we start by installing the agent:

In RHEL or CentOS:

.. code:: bash

    $ yum install cloudstack-agent

In Ubuntu:

.. code:: bash

    $ apt-get install cloudstack-agent

The host is now ready to be added to a cluster. This is covered in a
later section, see `Section 6.6, “Adding a Host” <#host-add>`__. It is
recommended that you continue to read the documentation before adding
the host!

Configure CPU model for KVM guest (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In additional,the CloudStack Agent allows host administrator to control
the guest CPU model which is exposed to KVM instances. By default, the
CPU model of KVM instance is likely QEMU Virtual CPU version x.x.x with
least CPU features exposed. There are a couple of reasons to specify the
CPU model:

-  

   To maximise performance of instances by exposing new host CPU
   features to the KVM instances;

-  

   To ensure a consistent default CPU across all machines,removing
   reliance of variable QEMU defaults;

For the most part it will be sufficient for the host administrator to
specify the guest CPU config in the per-host configuration file
(/etc/cloudstack/agent/agent.properties). This will be achieved by
introducing two new configuration parameters:

.. code::
 
    guest.cpu.mode=custom|host-model|host-passthrough
    guest.cpu.model=from /usr/share/libvirt/cpu_map.xml (only valid when guest.cpu.mode=custom)
    
    
                                                                

There are three choices to fulfill the cpu model changes:

#. 

   **custom:** you can explicitly specify one of the supported named
   model in /usr/share/libvirt/cpu\_map.xml

#. 

   **host-model:** libvirt will identify the CPU model in
   /usr/share/libvirt/cpu\_map.xml which most closely matches the host,
   and then request additional CPU flags to complete the match. This
   should give close to maximum functionality/performance, which
   maintaining good reliability/compatibility if the guest is migrated
   to another host with slightly different host CPUs.

#. 

   **host-passthrough:** libvirt will tell KVM to passthrough the host
   CPU with no modifications. The difference to host-model, instead of
   just matching feature flags, every last detail of the host CPU is
   matched. This gives absolutely best performance, and can be important
   to some apps which check low level CPU details, but it comes at a
   cost with respect to migration: the guest can only be migrated to an
   exactly matching host CPU.

Here are some examples:

-  

   custom

   .. code:: bash

       guest.cpu.mode=custom
       guest.cpu.model=SandyBridge

-  

   host-model

   .. code:: bash

       guest.cpu.mode=host-model

-  

   host-passthrough

   .. code:: bash

       guest.cpu.mode=host-passthrough

.. note:: host-passthrough may lead to migration failure,if you have this problem,you should use host-model or custom

Install and Configure libvirt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack uses libvirt for managing virtual machines. Therefore it is
vital that libvirt is configured correctly. Libvirt is a dependency of
cloudstack-agent and should already be installed.

#. 

   In order to have live migration working libvirt has to listen for
   unsecured TCP connections. We also need to turn off libvirts attempt
   to use Multicast DNS advertising. Both of these settings are in
   ``/etc/libvirt/libvirtd.conf``

   Set the following parameters:

   .. code:: bash

       listen_tls = 0

   .. code:: bash

       listen_tcp = 1

   .. code:: bash

       tcp_port = "16509"

   .. code:: bash

       auth_tcp = "none"

   .. code:: bash

       mdns_adv = 0

#. 

   Turning on "listen\_tcp" in libvirtd.conf is not enough, we have to
   change the parameters as well:

   On RHEL or CentOS modify ``/etc/sysconfig/libvirtd``:

   Uncomment the following line:

   .. code:: bash

       #LIBVIRTD_ARGS="--listen"

   On Ubuntu: modify ``/etc/default/libvirt-bin``

   Add "-l" to the following line

   .. code:: bash

       libvirtd_opts="-d"

   so it looks like:

   .. code:: bash

       libvirtd_opts="-d -l"

#. 

   In order to have the VNC Console work we have to make sure it will
   bind on 0.0.0.0. We do this by editing ``/etc/libvirt/qemu.conf``

   Make sure this parameter is set:

   .. code:: bash

       vnc_listen = "0.0.0.0"

#. 

   Restart libvirt

   In RHEL or CentOS:

   .. code:: bash

       $ service libvirtd restart

   In Ubuntu:

   .. code:: bash

       $ service libvirt-bin restart

Configure the Security Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack does various things which can be blocked by security
mechanisms like AppArmor and SELinux. These have to be disabled to
ensure the Agent has all the required permissions.

#. 

   Configure SELinux (RHEL and CentOS)

   #. 

      Check to see whether SELinux is installed on your machine. If not,
      you can skip this section.

      In RHEL or CentOS, SELinux is installed and enabled by default.
      You can verify this with:

      .. code:: bash

          $ rpm -qa | grep selinux

   #. 

      Set the SELINUX variable in ``/etc/selinux/config`` to
      "permissive". This ensures that the permissive setting will be
      maintained after a system reboot.

      In RHEL or CentOS:

      .. code:: bash

          vi /etc/selinux/config

      Change the following line

      .. code:: bash

          SELINUX=enforcing

      to this

      .. code:: bash

          SELINUX=permissive

   #. 

      Then set SELinux to permissive starting immediately, without
      requiring a system reboot.

      .. code:: bash

          $ setenforce permissive

#. 

   Configure Apparmor (Ubuntu)

   #. 

      Check to see whether AppArmor is installed on your machine. If
      not, you can skip this section.

      In Ubuntu AppArmor is installed and enabled by default. You can
      verify this with:

      .. code:: bash

          $ dpkg --list 'apparmor'

   #. 

      Disable the AppArmor profiles for libvirt

      .. code::

          $ ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/

      .. code::

          $ ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/

      .. code::

          $ apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd

      .. code::

          $ apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper

Configure the network bridges
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning:: This is a very important section, please make sure you read this thoroughly.

.. note:: This section details how to configure bridges using the native implementation in Linux. Please refer to the next section if you intend to use OpenVswitch

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

#. 

   VLAN 100 for management of the hypervisor

#. 

   VLAN 200 for public network of the instances (cloudbr0)

#. 

   VLAN 300 for private network of the instances (cloudbr1)

On VLAN 100 we give the Hypervisor the IP-Address 192.168.42.11/24 with
the gateway 192.168.42.1

.. note:: The Hypervisor and Management server don't have to be in the same subnet!

Configuring the network bridges
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It depends on the distribution you are using how to configure these,
below you'll find examples for RHEL/CentOS and Ubuntu.

.. note:: The goal is to have two bridges called 'cloudbr0' and 'cloudbr1' after this section. This should be used as a guideline only. The exact configuration will depend on your network layout.

Configure in RHEL or CentOS
'''''''''''''''''''''''''''

The required packages were installed when libvirt was installed, we can
proceed to configuring the network.

First we configure eth0

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0

Make sure it looks similar to:

.. code:: bash

    DEVICE=eth0
    HWADDR=00:04:xx:xx:xx:xx
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    TYPE=Ethernet

We now have to configure the three VLAN interfaces:

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0.100

.. code:: bash

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

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0.200

.. code:: bash

    DEVICE=eth0.200
    HWADDR=00:04:xx:xx:xx:xx
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    TYPE=Ethernet
    VLAN=yes
    BRIDGE=cloudbr0

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0.300

.. code:: bash

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

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr0

Now we just configure it is a plain bridge without an IP-Address

.. code:: bash

    DEVICE=cloudbr0
    TYPE=Bridge
    ONBOOT=yes
    BOOTPROTO=none
    IPV6INIT=no
    IPV6_AUTOCONF=no
    DELAY=5
    STP=yes

We do the same for cloudbr1

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr1

.. code:: bash

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

.. warning:: Make sure you have an alternative way like IPMI or ILO to reach the machine in case you made a configuration error and the network stops functioning!

Configure in Ubuntu
'''''''''''''''''''

All the required packages were installed when you installed libvirt, so
we only have to configure the network.

.. code:: bash

    vi /etc/network/interfaces

Modify the interfaces file to look like this:

.. code:: bash

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

.. warning:: Make sure you have an alternative way like IPMI or ILO to reach the machine in case you made a configuration error and the network stops functioning!

Configure the network using OpenVswitch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning:: This is a very important section, please make sure you read this thoroughly.

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

#. 

   VLAN 100 for management of the hypervisor

#. 

   VLAN 200 for public network of the instances (cloudbr0)

#. 

   VLAN 300 for private network of the instances (cloudbr1)

On VLAN 100 we give the Hypervisor the IP-Address 192.168.42.11/24 with
the gateway 192.168.42.1

.. note:: The Hypervisor and Management server don't have to be in the same subnet!

Configuring the network bridges
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It depends on the distribution you are using how to configure these,
below you'll find examples for RHEL/CentOS.

.. note:: The goal is to have three bridges called 'mgmt0', 'cloudbr0' and 'cloudbr1' after this section. This should be used as a guideline only. The exact configuration will depend on your network layout.

Configure OpenVswitch
'''''''''''''''''''''

The network interfaces using OpenVswitch are created using the ovs-vsctl
command. This command will configure the interfaces and persist them to
the OpenVswitch database.

First we create a main bridge connected to the eth0 interface. Next we
create three fake bridges, each connected to a specific vlan tag.

.. code:: bash

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

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0

Make sure it looks similar to:

.. code:: bash

    DEVICE=eth0
    HWADDR=00:04:xx:xx:xx:xx
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    TYPE=Ethernet

We have to configure the base bridge with the trunk.

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr

.. code:: bash

    DEVICE=cloudbr
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    DEVICETYPE=ovs
    TYPE=OVSBridge

We now have to configure the three VLAN bridges:

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-mgmt0

.. code:: bash

    DEVICE=mgmt0
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=static
    DEVICETYPE=ovs
    TYPE=OVSBridge
    IPADDR=192.168.42.11
    GATEWAY=192.168.42.1
    NETMASK=255.255.255.0

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr0

.. code:: bash

    DEVICE=cloudbr0
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    DEVICETYPE=ovs
    TYPE=OVSBridge

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr1

.. code:: bash

    DEVICE=cloudbr1
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    TYPE=OVSBridge
    DEVICETYPE=ovs

With this configuration you should be able to restart the network,
although a reboot is recommended to see if everything works properly.

.. warning:: Make sure you have an alternative way like IPMI or ILO to reach the machine in case you made a configuration error and the network stops functioning!

Configuring the firewall
~~~~~~~~~~~~~~~~~~~~~~~~

The hypervisor needs to be able to communicate with other hypervisors
and the management server needs to be able to reach the hypervisor.

In order to do so we have to open the following TCP ports (if you are
using a firewall):

#. 

   22 (SSH)

#. 

   1798

#. 

   16509 (libvirt)

#. 

   5900 - 6100 (VNC consoles)

#. 

   49152 - 49216 (libvirt live migration)

It depends on the firewall you are using how to open these ports. Below
you'll find examples how to open these ports in RHEL/CentOS and Ubuntu.

Open ports in RHEL/CentOS
^^^^^^^^^^^^^^^^^^^^^^^^^

RHEL and CentOS use iptables for firewalling the system, you can open
extra ports by executing the following iptable commands:

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 22 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 1798 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 16509 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 5900:6100 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 49152:49216 -j ACCEPT

These iptable settings are not persistent accross reboots, we have to
save them first.

.. code:: bash

    $ iptables-save > /etc/sysconfig/iptables

Open ports in Ubuntu
^^^^^^^^^^^^^^^^^^^^

The default firewall under Ubuntu is UFW (Uncomplicated FireWall), which
is a Python wrapper around iptables.

To open the required ports, execute the following commands:

.. code:: bash

    $ ufw allow proto tcp from any to any port 22

.. code:: bash

    $ ufw allow proto tcp from any to any port 1798

.. code:: bash

    $ ufw allow proto tcp from any to any port 16509

.. code:: bash

    $ ufw allow proto tcp from any to any port 5900:6100

.. code:: bash

    $ ufw allow proto tcp from any to any port 49152:49216

.. note:: By default UFW is not enabled on Ubuntu. Executing these commands with the firewall disabled does not enable the firewall.

Add the host to CloudStack
~~~~~~~~~~~~~~~~~~~~~~~~~~

The host is now ready to be added to a cluster. This is covered in a
later section, see `Section 6.6, “Adding a Host” <#host-add>`__. It is
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

Citrix XenServer Installation for CloudStack
--------------------------------------------

If you want to use the Citrix XenServer hypervisor to run guest virtual
machines, install XenServer 6.0 or XenServer 6.0.2 on the host(s) in
your cloud. For an initial installation, follow the steps below. If you
have previously installed XenServer and want to upgrade to another
version, see `Section 8.2.11, “Upgrading XenServer
Versions” <#xenserver-version-upgrading>`__.

System Requirements for XenServer Hosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  

   The host must be certified as compatible with one of the following.
   See the Citrix Hardware Compatibility Guide:
   `http://hcl.xensource.com <http://hcl.xensource.com>`__

   -  

      XenServer 5.6 SP2

   -  

      XenServer 6.0

   -  

      XenServer 6.0.2

-  

   You must re-install Citrix XenServer if you are going to re-use a
   host from a previous install.

-  

   Must support HVM (Intel-VT or AMD-V enabled)

-  

   Be sure all the hotfixes provided by the hypervisor vendor are
   applied. Track the release of hypervisor patches through your
   hypervisor vendor’s support channel, and apply patches as soon as
   possible after they are released. CloudStack will not track or notify
   you of required hypervisor patches. It is essential that your hosts
   are completely up to date with the provided hypervisor patches. The
   hypervisor vendor is likely to refuse to support any system that is
   not up to date with patches.

-  

   All hosts within a cluster must be homogeneous. The CPUs must be of
   the same type, count, and feature flags.

-  

   Must support HVM (Intel-VT or AMD-V enabled in BIOS)

-  

   64-bit x86 CPU (more cores results in better performance)

-  

   Hardware virtualization support required

-  

   4 GB of memory

-  

   36 GB of local disk

-  

   At least 1 NIC

-  

   Statically allocated IP Address

-  

   When you deploy CloudStack, the hypervisor host must not have any VMs
   already running

.. warning:: The lack of up-do-date hotfixes can lead to data corruption and lost VMs.

XenServer Installation Steps
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 

   From
   `https://www.citrix.com/English/ss/downloads/ <https://www.citrix.com/English/ss/downloads/>`__,
   download the appropriate version of XenServer for your CloudStack
   version (see `Section 8.2.1, “System Requirements for XenServer
   Hosts” <#system-requirements-xenserver-hosts>`__). Install it using
   the Citrix XenServer Installation Guide.

   Older Versions of XenServer:

   Note that you can download the most recent release of XenServer
   without having a Citrix account. If you wish to download older
   versions, you will need to create an account and look through the
   download archives.

Configure XenServer dom0 Memory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure the XenServer dom0 settings to allocate more memory to dom0.
This can enable XenServer to handle larger numbers of virtual machines.
We recommend 2940 MB of RAM for XenServer dom0. For instructions on how
to do this, see
`http://support.citrix.com/article/CTX126531 <http://support.citrix.com/article/CTX126531>`__.
The article refers to XenServer 5.6, but the same information applies to
XenServer 6.0.

Username and Password
~~~~~~~~~~~~~~~~~~~~~

All XenServers in a cluster must have the same username and password as
configured in CloudStack.

Time Synchronization
~~~~~~~~~~~~~~~~~~~~

The host must be set to use NTP. All hosts in a pod must have the same
time.

#. 

   Install NTP.

   .. code:: bash

       # yum install ntp

#. 

   Edit the NTP configuration file to point to your NTP server.

   .. code:: bash

       # vi /etc/ntp.conf

   Add one or more server lines in this file with the names of the NTP
   servers you want to use. For example:

   .. code:: bash

       server 0.xenserver.pool.ntp.org
       server 1.xenserver.pool.ntp.org
       server 2.xenserver.pool.ntp.org
       server 3.xenserver.pool.ntp.org

#. 

   Restart the NTP client.

   .. code:: bash

       # service ntpd restart

#. 

   Make sure NTP will start again upon reboot.

   .. code:: bash

       # chkconfig ntpd on


Install CloudStack XenServer Support Package (CSP)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(Optional)

To enable security groups, elastic load balancing, and elastic IP on
XenServer, download and install the CloudStack XenServer Support Package
(CSP). After installing XenServer, perform the following additional
steps on each XenServer host.

#. 

   Download the CSP software onto the XenServer host from one of the
   following links:

   For XenServer 6.0.2:

   `http://download.cloud.com/releases/3.0.1/XS-6.0.2/xenserver-cloud-supp.tgz <http://download.cloud.com/releases/3.0.1/XS-6.0.2/xenserver-cloud-supp.tgz>`__

   For XenServer 5.6 SP2:

   `http://download.cloud.com/releases/2.2.0/xenserver-cloud-supp.tgz <http://download.cloud.com/releases/2.2.0/xenserver-cloud-supp.tgz>`__

   For XenServer 6.0:

   `http://download.cloud.com/releases/3.0/xenserver-cloud-supp.tgz <http://download.cloud.com/releases/3.0/xenserver-cloud-supp.tgz>`__

#. 

   Extract the file:

   .. code:: bash

       # tar xf xenserver-cloud-supp.tgz

#. 

   Run the following script:

   .. code:: bash

       # xe-install-supplemental-pack xenserver-cloud-supp.iso

#. 

   If the XenServer host is part of a zone that uses basic networking,
   disable Open vSwitch (OVS):

   .. code:: bash

       # xe-switch-network-backend  bridge

   Restart the host machine when prompted.

The XenServer host is now ready to be added to CloudStack.

Primary Storage Setup for XenServer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack natively supports NFS, iSCSI and local storage. If you are
using one of these storage types, there is no need to create the
XenServer Storage Repository ("SR").

If, however, you would like to use storage connected via some other
technology, such as FiberChannel, you must set up the SR yourself. To do
so, perform the following steps. If you have your hosts in a XenServer
pool, perform the steps on the master node. If you are working with a
single XenServer which is not part of a cluster, perform the steps on
that XenServer.

#. 

   Connect FiberChannel cable to all hosts in the cluster and to the
   FiberChannel storage host.

#. 

   Rescan the SCSI bus. Either use the following command or use
   XenCenter to perform an HBA rescan.

   .. code:: bash

       # scsi-rescan

#. 

   Repeat step `2 <#rescan-scsi>`__ on every host.

#. 

   Check to be sure you see the new SCSI disk.

   .. code:: bash

       # ls /dev/disk/by-id/scsi-360a98000503365344e6f6177615a516b -l

   The output should look like this, although the specific file name
   will be different (scsi-<scsiID>):

   .. code:: bash

       lrwxrwxrwx 1 root root 9 Mar 16 13:47
       /dev/disk/by-id/scsi-360a98000503365344e6f6177615a516b -> ../../sdc

#. 

   Repeat step `4 <#verify-scsi>`__ on every host.

#. 

   On the storage server, run this command to get a unique ID for the
   new SR.

   .. code:: bash

       # uuidgen

   The output should look like this, although the specific ID will be
   different:

   .. code:: bash

       e6849e96-86c3-4f2c-8fcc-350cc711be3d

#. 

   Create the FiberChannel SR. In name-label, use the unique ID you just
   generated.

   .. code:: bash

       # xe sr-create type=lvmohba shared=true
       device-config:SCSIid=360a98000503365344e6f6177615a516b
       name-label="e6849e96-86c3-4f2c-8fcc-350cc711be3d"

   This command returns a unique ID for the SR, like the following
   example (your ID will be different):

   .. code:: bash

       7a143820-e893-6c6a-236e-472da6ee66bf

#. 

   To create a human-readable description for the SR, use the following
   command. In uuid, use the SR ID returned by the previous command. In
   name-description, set whatever friendly text you prefer.

   .. code:: bash

       # xe sr-param-set uuid=7a143820-e893-6c6a-236e-472da6ee66bf name-description="Fiber Channel storage repository"

   Make note of the values you will need when you add this storage to
   CloudStack later (see `Section 6.7, “Add Primary
   Storage” <#primary-storage-add>`__). In the Add Primary Storage
   dialog, in Protocol, you will choose PreSetup. In SR Name-Label, you
   will enter the name-label you set earlier (in this example,
   e6849e96-86c3-4f2c-8fcc-350cc711be3d).

#. 

   (Optional) If you want to enable multipath I/O on a FiberChannel SAN,
   refer to the documentation provided by the SAN vendor.

iSCSI Multipath Setup for XenServer (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When setting up the storage repository on a Citrix XenServer, you can
enable multipath I/O, which uses redundant physical components to
provide greater reliability in the connection between the server and the
SAN. To enable multipathing, use a SAN solution that is supported for
Citrix servers and follow the procedures in Citrix documentation. The
following links provide a starting point:

-  

   `http://support.citrix.com/article/CTX118791 <http://support.citrix.com/article/CTX118791>`__

-  

   `http://support.citrix.com/article/CTX125403 <http://support.citrix.com/article/CTX125403>`__

You can also ask your SAN vendor for advice about setting up your Citrix
repository for multipathing.

Make note of the values you will need when you add this storage to the
CloudStack later (see `Section 6.7, “Add Primary
Storage” <#primary-storage-add>`__). In the Add Primary Storage dialog,
in Protocol, you will choose PreSetup. In SR Name-Label, you will enter
the same name used to create the SR.

If you encounter difficulty, address the support team for the SAN
provided by your vendor. If they are not able to solve your issue, see
Contacting Support.

Physical Networking Setup for XenServer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once XenServer has been installed, you may need to do some additional
network configuration. At this point in the installation, you should
have a plan for what NICs the host will have and what traffic each NIC
will carry. The NICs should be cabled as necessary to implement your
plan.

If you plan on using NIC bonding, the NICs on all hosts in the cluster
must be cabled exactly the same. For example, if eth0 is in the private
bond on one host in a cluster, then eth0 must be in the private bond on
all hosts in the cluster.

The IP address assigned for the management network interface must be
static. It can be set on the host itself or obtained via static DHCP.

CloudStack configures network traffic of various types to use different
NICs or bonds on the XenServer host. You can control this process and
provide input to the Management Server through the use of XenServer
network name labels. The name labels are placed on physical interfaces
or bonds and configured in CloudStack. In some simple cases the name
labels are not required.

When configuring networks in a XenServer environment, network traffic
labels must be properly configured to ensure that the virtual interfaces
are created by CloudStack are bound to the correct physical device. The
name-label of the XenServer network must match the XenServer traffic
label specified while creating the CloudStack network. This is set by
running the following command:

.. code::

    xe network-param-set uuid=<network id> name-label=<CloudStack traffic label>

Configuring Public Network with a Dedicated NIC for XenServer (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CloudStack supports the use of a second NIC (or bonded pair of NICs,
described in `Section 8.2.10.4, “NIC Bonding for XenServer
(Optional)” <#xenserver-nic-bonding>`__) for the public network. If
bonding is not used, the public network can be on any NIC and can be on
different NICs on the hosts in a cluster. For example, the public
network can be on eth0 on node A and eth1 on node B. However, the
XenServer name-label for the public network must be identical across all
hosts. The following examples set the network label to "cloud-public".
After the management server is installed and running you must configure
it with the name of the chosen network label (e.g. "cloud-public"); this
is discussed in `Section 4.5, “Management Server
Installation” <#management-server-install-flow>`__.

If you are using two NICs bonded together to create a public network,
see `Section 8.2.10.4, “NIC Bonding for XenServer
(Optional)” <#xenserver-nic-bonding>`__.

If you are using a single dedicated NIC to provide public network
access, follow this procedure on each new host that is added to
CloudStack before adding the host.

#. 

   Run xe network-list and find the public network. This is usually
   attached to the NIC that is public. Once you find the network make
   note of its UUID. Call this <UUID-Public>.

#. 

   Run the following command.

   .. code:: bash

       # xe network-param-set name-label=cloud-public uuid=<UUID-Public>

Configuring Multiple Guest Networks for XenServer (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CloudStack supports the use of multiple guest networks with the
XenServer hypervisor. Each network is assigned a name-label in
XenServer. For example, you might have two networks with the labels
"cloud-guest" and "cloud-guest2". After the management server is
installed and running, you must add the networks and use these labels so
that CloudStack is aware of the networks.

Follow this procedure on each new host before adding the host to
CloudStack:

#. 

   Run xe network-list and find one of the guest networks. Once you find
   the network make note of its UUID. Call this <UUID-Guest>.

#. 

   Run the following command, substituting your own name-label and uuid
   values.

   .. code:: bash

       # xe network-param-set name-label=<cloud-guestN> uuid=<UUID-Guest>

#. 

   Repeat these steps for each additional guest network, using a
   different name-label and uuid each time.

Separate Storage Network for XenServer (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can optionally set up a separate storage network. This should be
done first on the host, before implementing the bonding steps below.
This can be done using one or two available NICs. With two NICs bonding
may be done as above. It is the administrator's responsibility to set up
a separate storage network.

Give the storage network a different name-label than what will be given
for other networks.

For the separate storage network to work correctly, it must be the only
interface that can ping the primary storage device's IP address. For
example, if eth0 is the management network NIC, ping -I eth0 <primary
storage device IP> must fail. In all deployments, secondary storage
devices must be pingable from the management network NIC or bond. If a
secondary storage device has been placed on the storage network, it must
also be pingable via the storage network NIC or bond on the hosts as
well.

You can set up two separate storage networks as well. For example, if
you intend to implement iSCSI multipath, dedicate two non-bonded NICs to
multipath. Each of the two networks needs a unique name-label.

If no bonding is done, the administrator must set up and name-label the
separate storage network on all hosts (masters and slaves).

Here is an example to set up eth5 to access a storage network on
172.16.0.0/24.

.. code::

    # xe pif-list host-name-label='hostname' device=eth5
    uuid(RO): ab0d3dd4-5744-8fae-9693-a022c7a3471d
    device ( RO): eth5
    
    #xe pif-reconfigure-ip DNS=172.16.3.3 gateway=172.16.0.1 IP=172.16.0.55 mode=static netmask=255.255.255.0 
        uuid=ab0d3dd4-5744-8fae-9693-a022c7a3471d

NIC Bonding for XenServer (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

XenServer supports Source Level Balancing (SLB) NIC bonding. Two NICs
can be bonded together to carry public, private, and guest traffic, or
some combination of these. Separate storage networks are also possible.
Here are some example supported configurations:

-  

   2 NICs on private, 2 NICs on public, 2 NICs on storage

-  

   2 NICs on private, 1 NIC on public, storage uses management network

-  

   2 NICs on private, 2 NICs on public, storage uses management network

-  

   1 NIC for private, public, and storage

All NIC bonding is optional.

XenServer expects all nodes in a cluster will have the same network
cabling and same bonds implemented. In an installation the master will
be the first host that was added to the cluster and the slave hosts will
be all subsequent hosts added to the cluster. The bonds present on the
master set the expectation for hosts added to the cluster later. The
procedure to set up bonds on the master and slaves are different, and
are described below. There are several important implications of this:

-  

   You must set bonds on the first host added to a cluster. Then you
   must use xe commands as below to establish the same bonds in the
   second and subsequent hosts added to a cluster.

-  

   Slave hosts in a cluster must be cabled exactly the same as the
   master. For example, if eth0 is in the private bond on the master, it
   must be in the management network for added slave hosts.

Management Network Bonding
''''''''''''''''''''''''''

The administrator must bond the management network NICs prior to adding
the host to CloudStack.

Creating a Private Bond on the First Host in the Cluster
''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use the following steps to create a bond in XenServer. These steps
should be run on only the first host in a cluster. This example creates
the cloud-private network with two physical NICs (eth0 and eth1) bonded
into it.

#. 

   Find the physical NICs that you want to bond together.

   .. code:: bash

       # xe pif-list host-name-label='hostname' device=eth0
       # xe pif-list host-name-label='hostname' device=eth1

   These command shows the eth0 and eth1 NICs and their UUIDs.
   Substitute the ethX devices of your choice. Call the UUID's returned
   by the above command slave1-UUID and slave2-UUID.

#. 

   Create a new network for the bond. For example, a new network with
   name "cloud-private".

   **This label is important. CloudStack looks for a network by a name
   you configure. You must use the same name-label for all hosts in the
   cloud for the management network.**

   .. code:: bash

       # xe network-create name-label=cloud-private
       # xe bond-create network-uuid=[uuid of cloud-private created above]
       pif-uuids=[slave1-uuid],[slave2-uuid]

Now you have a bonded pair that can be recognized by CloudStack as the
management network.

Public Network Bonding
''''''''''''''''''''''

Bonding can be implemented on a separate, public network. The
administrator is responsible for creating a bond for the public network
if that network will be bonded and will be separate from the management
network.

Creating a Public Bond on the First Host in the Cluster
'''''''''''''''''''''''''''''''''''''''''''''''''''''''

These steps should be run on only the first host in a cluster. This
example creates the cloud-public network with two physical NICs (eth2
and eth3) bonded into it.

#. 

   Find the physical NICs that you want to bond together.

   .. code:: bash

       #xe pif-list host-name-label='hostname' device=eth2
       # xe pif-list host-name-label='hostname' device=eth3

   These command shows the eth2 and eth3 NICs and their UUIDs.
   Substitute the ethX devices of your choice. Call the UUID's returned
   by the above command slave1-UUID and slave2-UUID.

#. 

   Create a new network for the bond. For example, a new network with
   name "cloud-public".

   **This label is important. CloudStack looks for a network by a name
   you configure. You must use the same name-label for all hosts in the
   cloud for the public network.**

   .. code:: bash

       # xe network-create name-label=cloud-public
       # xe bond-create network-uuid=[uuid of cloud-public created above]
       pif-uuids=[slave1-uuid],[slave2-uuid]

Now you have a bonded pair that can be recognized by CloudStack as the
public network.

Adding More Hosts to the Cluster
''''''''''''''''''''''''''''''''

With the bonds (if any) established on the master, you should add
additional, slave hosts. Run the following command for all additional
hosts to be added to the cluster. This will cause the host to join the
master in a single XenServer pool.

.. code:: bash

    # xe pool-join master-address=[master IP] master-username=root
    master-password=[your password]

Complete the Bonding Setup Across the Cluster
'''''''''''''''''''''''''''''''''''''''''''''

With all hosts added to the pool, run the cloud-setup-bond script. This
script will complete the configuration and set up of the bonds across
all hosts in the cluster.

#. 

   Copy the script from the Management Server in
   /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver/cloud-setup-bonding.sh
   to the master host and ensure it is executable.

#. 

   Run the script:

   .. code:: bash

       # ./cloud-setup-bonding.sh

Now the bonds are set up and configured properly across the cluster.

Upgrading XenServer Versions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section tells how to upgrade XenServer software on CloudStack
hosts. The actual upgrade is described in XenServer documentation, but
there are some additional steps you must perform before and after the
upgrade.

.. note:: Be sure the hardware is certified compatible with the new version of XenServer.

To upgrade XenServer:

#. 

   Upgrade the database. On the Management Server node:

   #. 

      Back up the database:

      .. code:: bash

          # mysqldump --user=root --databases cloud > cloud.backup.sql
          # mysqldump --user=root --databases cloud_usage > cloud_usage.backup.sql

   #. 

      You might need to change the OS type settings for VMs running on
      the upgraded hosts.

      -  

         If you upgraded from XenServer 5.6 GA to XenServer 5.6 SP2,
         change any VMs that have the OS type CentOS 5.5 (32-bit),
         Oracle Enterprise Linux 5.5 (32-bit), or Red Hat Enterprise
         Linux 5.5 (32-bit) to Other Linux (32-bit). Change any VMs that
         have the 64-bit versions of these same OS types to Other Linux
         (64-bit).

      -  

         If you upgraded from XenServer 5.6 SP2 to XenServer 6.0.2,
         change any VMs that have the OS type CentOS 5.6 (32-bit),
         CentOS 5.7 (32-bit), Oracle Enterprise Linux 5.6 (32-bit),
         Oracle Enterprise Linux 5.7 (32-bit), Red Hat Enterprise Linux
         5.6 (32-bit) , or Red Hat Enterprise Linux 5.7 (32-bit) to
         Other Linux (32-bit). Change any VMs that have the 64-bit
         versions of these same OS types to Other Linux (64-bit).

      -  

         If you upgraded from XenServer 5.6 to XenServer 6.0.2, do all
         of the above.

   #. 

      Restart the Management Server and Usage Server. You only need to
      do this once for all clusters.

      .. code:: bash

          # service cloudstack-management start
          # service cloudstack-usage start

#. 

   Disconnect the XenServer cluster from CloudStack.

   #. 

      Log in to the CloudStack UI as root.

   #. 

      Navigate to the XenServer cluster, and click Actions – Unmanage.

   #. 

      Watch the cluster status until it shows Unmanaged.

#. 

   Log in to one of the hosts in the cluster, and run this command to
   clean up the VLAN:

   .. code:: bash

       # . /opt/xensource/bin/cloud-clean-vlan.sh

#. 

   Still logged in to the host, run the upgrade preparation script:

   .. code:: bash

       # /opt/xensource/bin/cloud-prepare-upgrade.sh

   Troubleshooting: If you see the error "can't eject CD," log in to the
   VM and umount the CD, then run the script again.

#. 

   Upgrade the XenServer software on all hosts in the cluster. Upgrade
   the master first.

   #. 

      Live migrate all VMs on this host to other hosts. See the
      instructions for live migration in the Administrator's Guide.

      Troubleshooting: You might see the following error when you
      migrate a VM:

      .. code::

          [root@xenserver-qa-2-49-4 ~]# xe vm-migrate live=true host=xenserver-qa-2-49-5 vm=i-2-8-VM
          You attempted an operation on a VM which requires PV drivers to be installed but the drivers were not detected.
          vm: b6cf79c8-02ee-050b-922f-49583d9f1a14 (i-2-8-VM)

      To solve this issue, run the following:

      .. code:: bash

          # /opt/xensource/bin/make_migratable.sh  b6cf79c8-02ee-050b-922f-49583d9f1a14

   #. 

      Reboot the host.

   #. 

      Upgrade to the newer version of XenServer. Use the steps in
      XenServer documentation.

   #. 

      After the upgrade is complete, copy the following files from the
      management server to this host, in the directory locations shown
      below:

      =================================================================================   =======================================
      Copy this Management Server file                                                    To this location on the XenServer host
      =================================================================================   =======================================
      /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver/xenserver60/NFSSR.py   /opt/xensource/sm/NFSSR.py
      /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver/setupxenserver.sh      /opt/xensource/bin/setupxenserver.sh
      /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver/make\_migratable.sh    /opt/xensource/bin/make\_migratable.sh
      /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver/cloud-clean-vlan.sh    /opt/xensource/bin/cloud-clean-vlan.sh
      =================================================================================   =======================================

   #. 

      Run the following script:

      .. code:: bash

          # /opt/xensource/bin/setupxenserver.sh

      Troubleshooting: If you see the following error message, you can
      safely ignore it.

      .. code:: bash

          mv: cannot stat `/etc/cron.daily/logrotate`: No such file or directory

   #. 

      Plug in the storage repositories (physical block devices) to the
      XenServer host:

      .. code:: bash

          # for pbd in `xe pbd-list currently-attached=false| grep ^uuid | awk '{print $NF}'`; do xe pbd-plug uuid=$pbd ; done

      .. note:: If you add a host to this XenServer pool, you need to migrate all VMs on this host to other hosts, and eject this host from XenServer pool.

#. 

   Repeat these steps to upgrade every host in the cluster to the same
   version of XenServer.

#. 

   Run the following command on one host in the XenServer cluster to
   clean up the host tags:

   .. code:: bash

       # for host in $(xe host-list | grep ^uuid | awk '{print $NF}') ; do xe host-param-clear uuid=$host param-name=tags; done;

   .. note:: When copying and pasting a command, be sure the command has pasted as
   a single line before executing. Some document viewers may introduce
   unwanted line breaks in copied text.

#. 

   Reconnect the XenServer cluster to CloudStack.

   #. 

      Log in to the CloudStack UI as root.

   #. 

      Navigate to the XenServer cluster, and click Actions – Manage.

   #. 

      Watch the status to see that all the hosts come up.

#. 

   After all hosts are up, run the following on one host in the cluster:

   .. code:: bash

       # /opt/xensource/bin/cloud-clean-vlan.sh

Installing Hyper-V for CloudStack
---------------------------------

If you want to use Hyper-V hypervisor to run guest virtual machines,
install Hyper-V on the hosts in your cloud. The instructions in this
section doesn't duplicate Hyper-V Installation documentation. It
provides the CloudStack-specific steps that are needed to prepare a
Hyper-V host to work with CloudStack.

System Requirements for Hyper-V Hypervisor Hosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Supported Operating Systems for Hyper-V Hosts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  

   Windows Server 2012 R2 Standard

-  

   Windows Server 2012 R2 Datacenter

-  

   Hyper-V 2012 R2

Minimum System Requirements for Hyper-V Hosts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  

   1.4 GHz 64-bit processor with hardware-assisted virtualization.

-  

   800 MB of RAM

-  

   32 GB of disk space

-  

   Gigabit (10/100/1000baseT) Ethernet adapter

Supported Storage
^^^^^^^^^^^^^^^^^

-  

   Primary Storage: Server Message Block (SMB) Version 3, Local

-  

   Secondary Storage: SMB

Preparation Checklist for Hyper-V
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a smoother installation, gather the following information before you
start:

Hyper-V Requirements

Value

Description

Server Roles

Hyper-V

After the Windows Server 2012 R2 installation, ensure that Hyper-V is
selected from Server Roles. For more information, see `Installing
Hyper-V <http://technet.microsoft.com/en-us/library/jj134187.aspx#BKMK_Step2>`__.

Share Location

New folders in the /Share directory

Ensure that folders are created for Primary and Secondary storage. The
SMB share and the hosts should be part of the same domain.

If you are using Windows SMB share, the location of the file share for
the Hyper-V deployment will be the new folder created in the \\Shares on
the selected volume. You can create sub-folders for both CloudStack
Primary and Secondary storage within the share location. When you select
the profile for the file shares, ensure that you select SMB Share
-Applications. This creates the file shares with settings appropriate
for Hyper-V.

Domain and Hosts

Hosts should be part of the same Active Directory domain.

Hyper-V Users

Full control

Full control on the SMB file share.

Virtual Switch

If you are using Hyper-V 2012 R2, manually create an external virtual
switch before adding the host to CloudStack. If the Hyper-V host is
added to the Hyper-V manager, select the host, then click Virtual Switch
Manager, then New Virtual Switch. In the External Network, select the
desired NIC adapter and click Apply.

If you are using Windows 2012 R2, virtual switch is created
automatically.

Virtual Switch Name

Take a note of the name of the virtual switch. You need to specify that
when configuring CloudStack physical network labels.

Hyper-V Domain Users

-  

   Add the Hyper-V domain users to the Hyper-V Administrators group.

-  

   A domain user should have full control on the SMB share that is
   exported for primary and secondary storage.

-  

   This domain user should be part of the Hyper-V Administrators and
   Local Administrators group on the Hyper-V hosts that are to be
   managed by CloudStack.

-  

   The Hyper-V Agent service runs with the credentials of this domain
   user account.

-  

   Specify the credential of the domain user while adding a host to
   CloudStack so that it can manage it.

-  

   Specify the credential of the domain user while adding a shared SMB
   primary or secondary storage.

Migration

Migration

Enable Migration.

For more information, see `Configuring Live
Migration <http://technet.microsoft.com/en-us/library/jj134199.aspx%20>`__.

Migration

Delegation

If you want to use Live Migration, enable Delegation. Enable the
following services of other hosts participating in Live Migration: CIFS
and Microsoft Virtual System Migration Service.

Migration

Kerberos

Enable Kerberos for Live Migration.

Network Access Permission for Dial-in

Allow access

Allow access for Dial-in connections.

Hyper-V Installation Steps
~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 

   Download the operating system from `Windows Server 2012
   R2 <http://technet.microsoft.com/en-us/windowsserver/hh534429>`__ .

#. 

   Install it on the host as given in `Install and Deploy Windows Server
   2012 R2 <http://technet.microsoft.com/library/hh831620>`__.

#. 

   Post installation, ensure that you enable Hyper-V role in the server.

#. 

   If no Active Directory domain exists in your deployment, create one
   and add users to the domain.

#. 

   In the Active Directory domain, ensure that all the Hyper-v hosts are
   added so that all the hosts are part of the domain.

#. 

   Add the domain user to the following groups on the Hyper-V host:
   Hyper-V Administrators and Local Administrators.


Installing the CloudStack Agent on a Hyper-V Host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Hyper-V Agent helps CloudStack perform operations on the Hyper-V
hosts. This Agent communicates with the Management Server and controls
all the instances on the host. Each Hyper-V host must have the Hyper-V
Agent installed on it for successful interaction between the host and
CloudStack. The Hyper-V Agent runs as a Windows service. Install the
Agent on each host using the following steps.

CloudStack Management Server communicates with Hyper-V Agent by using
HTTPS. For secure communication between the Management Server and the
host, install a self-signed certificate on port 8250.

.. note:: The Agent installer automatically perform this operation. You have not selected this option during the Agent installation, it can also be done manually as given in step 1.

#. 

   Create and add a self-signed SSL certificate on port 8250:

   #. 

      Create A self-signed SSL certificate:

      .. code:: bash

          #  New-SelfSignedCertificate -DnsName apachecloudstack -CertStoreLocation Cert:\LocalMachine\My

      This command creates the self-signed certificate and add that to
      the certificate store ``LocalMachine\My``.

   #. 

      Add the created certificate to port 8250 for https communication:

      .. code::

          netsh http add sslcert ipport=0.0.0.0:8250 certhash=<thumbprint> appid="{727beb1c-6e7c-49b2-8fbd-f03dbe481b08}"

      Thumbprint is the thumbprint of the certificate you created.

#. 

   Build the CloudStack Agent for Hyper-V as given in `Building
   CloudStack Hyper-V
   Agent <https://cwiki.apache.org/confluence/display/CLOUDSTACK/Creating+Hyperv+Agent+Installer>`__.

#. 

   As an administrator, run the installer.

#. 

   Provide the Hyper-V admin credentials when prompted.

   When the agent installation is finished, the agent runs as a service
   on the host machine.

Physical Network Configuration for Hyper-V
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You should have a plan for how the hosts will be cabled and which
physical NICs will carry what types of traffic. By default, CloudStack
will use the device that is used for the default route.

If you are using Hyper-V 2012 R2, manually create an external virtual
switch before adding the host to CloudStack. If the Hyper-V host is
added to the Hyper-V manager, select the host, then click Virtual Switch
Manager, then New Virtual Switch. In the External Network, select the
desired NIC adapter and click Apply.

If you are using Windows 2012 R2, virtual switch is created
automatically.

Storage Preparation for Hyper-V (Optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack allows administrators to set up shared Primary Storage and
Secondary Storage that uses SMB.

#. 

   Create a SMB storage and expose it over SMB Version 3.

   For more information, see `Deploying Hyper-V over
   SMB <http://technet.microsoft.com/en-us/library/jj134187.aspx>`__.

   You can also create and export SMB share using Windows. After the
   Windows Server 2012 R2 installation, select File and Storage Services
   from Server Roles to create an SMB file share. For more information,
   see `Creating an SMB File Share Using Server
   Manager <http://technet.microsoft.com/en-us/library/jj134187.aspx#BKMK_Step3>`__.

#. 

   Add the SMB share to the Active Directory domain.

   The SMB share and the hosts managed by CloudStack need to be in the
   same domain. However, the storage should be accessible from the
   Management Server with the domain user privileges.

#. 

   While adding storage to CloudStack, ensure that the correct domain,
   and credentials are supplied. This user should be able to access the
   storage from the Management Server.

VMware vSphere Installation and Configuration
---------------------------------------------

If you want to use the VMware vSphere hypervisor to run guest virtual
machines, install vSphere on the host(s) in your cloud.

System Requirements for vSphere Hosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Software requirements:
^^^^^^^^^^^^^^^^^^^^^^

-  

   vSphere and vCenter, both version 4.1 or 5.0.

   vSphere Standard is recommended. Note however that customers need to
   consider the CPU constraints in place with vSphere licensing. See
   `http://www.vmware.com/files/pdf/vsphere\_pricing.pdf <http://www.vmware.com/files/pdf/vsphere_pricing.pdf>`__
   and discuss with your VMware sales representative.

   vCenter Server Standard is recommended.

-  

   Be sure all the hotfixes provided by the hypervisor vendor are
   applied. Track the release of hypervisor patches through your
   hypervisor vendor's support channel, and apply patches as soon as
   possible after they are released. CloudStack will not track or notify
   you of required hypervisor patches. It is essential that your hosts
   are completely up to date with the provided hypervisor patches. The
   hypervisor vendor is likely to refuse to support any system that is
   not up to date with patches.

.. warning:: Apply All Necessary Hotfixes. The lack of up-do-date hotfixes can lead to data corruption and lost VMs.

Hardware requirements:
^^^^^^^^^^^^^^^^^^^^^^

-  

   The host must be certified as compatible with vSphere. See the VMware
   Hardware Compatibility Guide at
   `http://www.vmware.com/resources/compatibility/search.php <http://www.vmware.com/resources/compatibility/search.php>`__.

-  

   All hosts must be 64-bit and must support HVM (Intel-VT or AMD-V
   enabled).

-  

   All hosts within a cluster must be homogenous. That means the CPUs
   must be of the same type, count, and feature flags.

-  

   64-bit x86 CPU (more cores results in better performance)

-  

   Hardware virtualization support required

-  

   4 GB of memory

-  

   36 GB of local disk

-  

   At least 1 NIC

-  

   Statically allocated IP Address

vCenter Server requirements:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  

   Processor - 2 CPUs 2.0GHz or higher Intel or AMD x86 processors.
   Processor requirements may be higher if the database runs on the same
   machine.

-  

   Memory - 3GB RAM. RAM requirements may be higher if your database
   runs on the same machine.

-  

   Disk storage - 2GB. Disk requirements may be higher if your database
   runs on the same machine.

-  

   Microsoft SQL Server 2005 Express disk requirements. The bundled
   database requires up to 2GB free disk space to decompress the
   installation archive.

-  

   Networking - 1Gbit or 10Gbit.

For more information, see "vCenter Server and the vSphere Client
Hardware Requirements" at
`http://pubs.vmware.com/vsp40/wwhelp/wwhimpl/js/html/wwhelp.htm#href=install/c\_vc\_hw.html <http://pubs.vmware.com/vsp40/wwhelp/wwhimpl/js/html/wwhelp.htm#href=install/c_vc_hw.html>`__.

Other requirements:
^^^^^^^^^^^^^^^^^^^

-  

   VMware vCenter Standard Edition 4.1 or 5.0 must be installed and
   available to manage the vSphere hosts.

-  

   vCenter must be configured to use the standard port 443 so that it
   can communicate with the CloudStack Management Server.

-  

   You must re-install VMware ESXi if you are going to re-use a host
   from a previous install.

-  

   CloudStack requires VMware vSphere 4.1 or 5.0. VMware vSphere 4.0 is
   not supported.

-  

   All hosts must be 64-bit and must support HVM (Intel-VT or AMD-V
   enabled). All hosts within a cluster must be homogeneous. That means
   the CPUs must be of the same type, count, and feature flags.

-  

   The CloudStack management network must not be configured as a
   separate virtual network. The CloudStack management network is the
   same as the vCenter management network, and will inherit its
   configuration. See `Section 8.4.5.2, “Configure vCenter Management
   Network” <#vmware-physical-host-networking-config-vcenter-mgt>`__.

-  

   CloudStack requires ESXi. ESX is not supported.

-  

   All resources used for CloudStack must be used for CloudStack only.
   CloudStack cannot share instance of ESXi or storage with other
   management consoles. Do not share the same storage volumes that will
   be used by CloudStack with a different set of ESXi servers that are
   not managed by CloudStack.

-  

   Put all target ESXi hypervisors in a cluster in a separate Datacenter
   in vCenter.

-  

   The cluster that will be managed by CloudStack should not contain any
   VMs. Do not run the management server, vCenter or any other VMs on
   the cluster that is designated for CloudStack use. Create a separate
   cluster for use of CloudStack and make sure that they are no VMs in
   this cluster.

-  

   All the required VLANS must be trunked into all network switches that
   are connected to the ESXi hypervisor hosts. These would include the
   VLANS for Management, Storage, vMotion, and guest VLANs. The guest
   VLAN (used in Advanced Networking; see Network Setup) is a contiguous
   range of VLANs that will be managed by CloudStack.

Preparation Checklist for VMware
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a smoother installation, gather the following information before you
start:

-  

   Information listed in `Section 8.4.2.1, “vCenter
   Checklist” <#vmware-vcenter-checklist>`__

-  

   Information listed in `Section 8.4.2.2, “Networking Checklist for
   VMware” <#vmware-network-checklist>`__

vCenter Checklist
^^^^^^^^^^^^^^^^^

You will need the following information about vCenter.

========================  =====================================
vCenter Requirement       Notes
========================  =====================================
vCenter User              This user must have admin privileges.
vCenter User Password     Password for the above user.
vCenter Datacenter Name   Name of the datacenter.
vCenter Cluster Name      Name of the cluster.
========================  =====================================

Networking Checklist for VMware
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You will need the following information about VLAN.

============================  ==========================================================================================
VLAN Information              Notes
============================  ==========================================================================================
ESXi VLAN                     VLAN on which all your ESXi hypervisors reside.
ESXI VLAN IP Address          IP Address Range in the ESXi VLAN. One address per Virtual Router is used from this range.
ESXi VLAN IP Gateway
ESXi VLAN Netmask
Management Server VLAN        VLAN on which the CloudStack Management server is installed.
Public VLAN                   VLAN for the Public Network.
Public VLAN Gateway
Public VLAN Netmask
Public VLAN IP Address Range  Range of Public IP Addresses available for CloudStack use. These
                              addresses will be used for virtual router on CloudStack to route private
                              traffic to external networks.
VLAN Range for Customer use   A contiguous range of non-routable VLANs. One VLAN will be assigned for
                              each customer.
============================  ==========================================================================================


vSphere Installation Steps
~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 

   If you haven't already, you'll need to download and purchase vSphere
   from the VMware Website
   (`https://www.vmware.com/tryvmware/index.php?p=vmware-vsphere&lp=1 <https://www.vmware.com/tryvmware/index.php?p=vmware-vsphere&lp=1>`__)
   and install it by following the VMware vSphere Installation Guide.

#. 

   Following installation, perform the following configuration, which
   are described in the next few sections:

   ====================================================================================================== ===================
   Required                                                                                                Optional
   ====================================================================================================== ===================
   ESXi host setup                                                                                         NIC bonding
   Configure host physical networking,virtual switch, vCenter Management Network, and extended port range  Multipath storage
   Prepare storage for iSCSI
   Configure clusters in vCenter and add hosts to them, or add hosts without clusters to vCenter
   ====================================================================================================== ===================

ESXi Host setup
~~~~~~~~~~~~~~~

All ESXi hosts should enable CPU hardware virtualization support in
BIOS. Please note hardware virtualization support is not enabled by
default on most servers.

Physical Host Networking
~~~~~~~~~~~~~~~~~~~~~~~~

You should have a plan for cabling the vSphere hosts. Proper network
configuration is required before adding a vSphere host to CloudStack. To
configure an ESXi host, you can use vClient to add it as standalone host
to vCenter first. Once you see the host appearing in the vCenter
inventory tree, click the host node in the inventory tree, and navigate
to the Configuration tab.

|vspherephysicalnetwork.png: vSphere client|

In the host configuration tab, click the "Hardware/Networking" link to
bring up the networking configuration page as above.

Configure Virtual Switch
^^^^^^^^^^^^^^^^^^^^^^^^

A default virtual switch vSwitch0 is created. CloudStack requires all
ESXi hosts in the cloud to use the same set of virtual switch names. If
you change the default virtual switch name, you will need to configure
one or more CloudStack configuration variables as well.

Separating Traffic
''''''''''''''''''

CloudStack allows you to use vCenter to configure three separate
networks per ESXi host. These networks are identified by the name of the
vSwitch they are connected to. The allowed networks for configuration
are public (for traffic to/from the public internet), guest (for
guest-guest traffic), and private (for management and usually storage
traffic). You can use the default virtual switch for all three, or
create one or two other vSwitches for those traffic types.

If you want to separate traffic in this way you should first create and
configure vSwitches in vCenter according to the vCenter instructions.
Take note of the vSwitch names you have used for each traffic type. You
will configure CloudStack to use these vSwitches.

Increasing Ports
''''''''''''''''

By default a virtual switch on ESXi hosts is created with 56 ports. We
recommend setting it to 4088, the maximum number of ports allowed. To do
that, click the "Properties..." link for virtual switch (note this is
not the Properties link for Networking).

|vsphereincreaseports.png: vSphere client|

In vSwitch properties dialog, select the vSwitch and click Edit. You
should see the following dialog:

|vspherevswitchproperties.png: vSphere client|

In this dialog, you can change the number of switch ports. After you've
done that, ESXi hosts are required to reboot in order for the setting to
take effect.

Configure vCenter Management Network
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the vSwitch properties dialog box, you may see a vCenter management
network. This same network will also be used as the CloudStack
management network. CloudStack requires the vCenter management network
to be configured properly. Select the management network item in the
dialog, then click Edit.

|vspheremgtnetwork.png: vSphere client|

Make sure the following values are set:

-  

   VLAN ID set to the desired ID

-  

   vMotion enabled.

-  

   Management traffic enabled.

If the ESXi hosts have multiple VMKernel ports, and ESXi is not using
the default value "Management Network" as the management network name,
you must follow these guidelines to configure the management network
port group so that CloudStack can find it:

-  

   Use one label for the management network port across all ESXi hosts.

-  

   In the CloudStack UI, go to Configuration - Global Settings and set
   vmware.management.portgroup to the management network label from the
   ESXi hosts.

Extend Port Range for CloudStack Console Proxy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

(Applies only to VMware vSphere version 4.x)

You need to extend the range of firewall ports that the console proxy
works with on the hosts. This is to enable the console proxy to work
with VMware-based VMs. The default additional port range is 59000-60000.
To extend the port range, log in to the VMware ESX service console on
each host and run the following commands:

.. code:: bash

    esxcfg-firewall -o 59000-60000,tcp,in,vncextras
    esxcfg-firewall -o 59000-60000,tcp,out,vncextras

Configure NIC Bonding for vSphere
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

NIC bonding on vSphere hosts may be done according to the vSphere
installation guide.

Configuring a vSphere Cluster with Nexus 1000v Virtual Switch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack supports Cisco Nexus 1000v dvSwitch (Distributed Virtual
Switch) for virtual network configuration in a VMware vSphere
environment. This section helps you configure a vSphere cluster with
Nexus 1000v virtual switch in a VMware vCenter environment. For
information on creating a vSphere cluster, see `Section 8.4, “VMware
vSphere Installation and Configuration” <#vmware-install>`__

About Cisco Nexus 1000v Distributed Virtual Switch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Cisco Nexus 1000V virtual switch is a software-based virtual machine
access switch for VMware vSphere environments. It can span multiple
hosts running VMware ESXi 4.0 and later. A Nexus virtual switch consists
of two components: the Virtual Supervisor Module (VSM) and the Virtual
Ethernet Module (VEM). The VSM is a virtual appliance that acts as the
switch's supervisor. It controls multiple VEMs as a single network
device. The VSM is installed independent of the VEM and is deployed in
redundancy mode as pairs or as a standalone appliance. The VEM is
installed on each VMware ESXi server to provide packet-forwarding
capability. It provides each virtual machine with dedicated switch
ports. This VSM-VEM architecture is analogous to a physical Cisco
switch's supervisor (standalone or configured in high-availability mode)
and multiple linecards architecture.

Nexus 1000v switch uses vEthernet port profiles to simplify network
provisioning for virtual machines. There are two types of port profiles:
Ethernet port profile and vEthernet port profile. The Ethernet port
profile is applied to the physical uplink ports-the NIC ports of the
physical NIC adapter on an ESXi server. The vEthernet port profile is
associated with the virtual NIC (vNIC) that is plumbed on a guest VM on
the ESXi server. The port profiles help the network administrators
define network policies which can be reused for new virtual machines.
The Ethernet port profiles are created on the VSM and are represented as
port groups on the vCenter server.

Prerequisites and Guidelines
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section discusses prerequisites and guidelines for using Nexus
virtual switch in CloudStack. Before configuring Nexus virtual switch,
ensure that your system meets the following requirements:

-  

   A cluster of servers (ESXi 4.1 or later) is configured in the
   vCenter.

-  

   Each cluster managed by CloudStack is the only cluster in its vCenter
   datacenter.

-  

   A Cisco Nexus 1000v virtual switch is installed to serve the
   datacenter that contains the vCenter cluster. This ensures that
   CloudStack doesn't have to deal with dynamic migration of virtual
   adapters or networks across other existing virtual switches. See
   `Cisco Nexus 1000V Installation and Upgrade
   Guide <http://www.cisco.com/en/US/docs/switches/datacenter/nexus1000/sw/4_2_1_s_v_1_5_1/install_upgrade/vsm_vem/guide/n1000v_installupgrade.html>`__
   for guidelines on how to install the Nexus 1000v VSM and VEM modules.

-  

   The Nexus 1000v VSM is not deployed on a vSphere host that is managed
   by CloudStack.

-  

   When the maximum number of VEM modules per VSM instance is reached,
   an additional VSM instance is created before introducing any more
   ESXi hosts. The limit is 64 VEM modules for each VSM instance.

-  

   CloudStack expects that the Management Network of the ESXi host is
   configured on the standard vSwitch and searches for it in the
   standard vSwitch. Therefore, ensure that you do not migrate the
   management network to Nexus 1000v virtual switch during
   configuration.

-  

   All information given in `Section 8.4.6.3, “Nexus 1000v Virtual
   Switch
   Preconfiguration” <#vmware-vsphere-cluster-config-nexus-vswitch-preconfig>`__

Nexus 1000v Virtual Switch Preconfiguration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Preparation Checklist
'''''''''''''''''''''

For a smoother configuration of Nexus 1000v switch, gather the following
information before you start:

-  

   vCenter credentials

-  

   Nexus 1000v VSM IP address

-  

   Nexus 1000v VSM Credentials

-  

   Ethernet port profile names

vCenter Credentials Checklist
'''''''''''''''''''''''''''''                                          

You will need the following information about vCenter:

=============================  =========  =============================================================================
Nexus vSwitch Requirements     Value      Notes
=============================  =========  =============================================================================
vCenter IP                                The IP address of the vCenter.
Secure HTTP Port Number        443        Port 443 is configured by default; however, you can change the port if needed.
vCenter User ID                           The vCenter user with administrator-level privileges. The vCenter User ID is 
                                          required when you configure the virtual switch in CloudStack.
vCenter Password                          The password for the vCenter user specified above. The password for this
                                          vCenter user is required when you configure the switch in CloudStack.
=============================  =========  =============================================================================


Network Configuration Checklist
'''''''''''''''''''''''''''''''                                            

The following information specified in the Nexus Configure Networking
screen is displayed in the Details tab of the Nexus dvSwitch in the
CloudStack UI:

**Control Port Group VLAN ID**
                        The VLAN ID of the Control Port Group. The control VLAN is used for communication between the VSM and the VEMs.

**Management Port Group VLAN ID**
                        The VLAN ID of the Management Port Group. The management VLAN corresponds to the mgmt0 interface that is used to establish and maintain the connection between the VSM and VMware vCenter Server.

**Packet Port Group VLAN ID**
                        The VLAN ID of the Packet Port Group. The packet VLAN forwards relevant data packets from the VEMs to the VSM.

.. note:: The VLANs used for control, packet, and management port groups can be the same.

For more information, see `Cisco Nexus 1000V Getting Started
Guide <http://www.cisco.com/en/US/docs/switches/datacenter/nexus1000/sw/4_2_1_s_v_1_4_b/getting_started/configuration/guide/n1000v_gsg.pdf>`__.

VSM Configuration Checklist
'''''''''''''''''''''''''''                                        

You will need the following VSM configuration parameters:

**Admin Name and Password**
                       The admin name and password to connect to the VSM appliance. You must specify these credentials while configuring Nexus virtual switch.
**Management IP Address**
                       This is the IP address of the VSM appliance. This is the IP address you specify in the virtual switch IP Address field while configuting Nexus virtual switch.
**SSL**
                       Should be set to Enable.Always enable SSL. SSH is usually enabled by default during the VSM
installation. However, check whether the SSH connection to the VSM is
working, without which CloudStack failes to connect to the VSM.

Creating a Port Profile
'''''''''''''''''''''''

-  

   Whether you create a Basic or Advanced zone configuration, ensure
   that you always create an Ethernet port profile on the VSM after you
   install it and before you create the zone.

   -  

      The Ethernet port profile created to represent the physical
      network or networks used by an Advanced zone configuration trunk
      all the VLANs including guest VLANs, the VLANs that serve the
      native VLAN, and the packet/control/data/management VLANs of the
      VSM.

   -  

      The Ethernet port profile created for a Basic zone configuration
      does not trunk the guest VLANs because the guest VMs do not get
      their own VLANs provisioned on their network interfaces in a Basic
      zone.

-  

   An Ethernet port profile configured on the Nexus 1000v virtual switch
   should not use in its set of system VLANs, or any of the VLANs
   configured or intended to be configured for use towards VMs or VM
   resources in the CloudStack environment.

-  

   You do not have to create any vEthernet port profiles – CloudStack
   does that during VM deployment.

-  

   Ensure that you create required port profiles to be used by
   CloudStack for different traffic types of CloudStack, such as
   Management traffic, Guest traffic, Storage traffic, and Public
   traffic. The physical networks configured during zone creation should
   have a one-to-one relation with the Ethernet port profiles.

|vmwarenexusportprofile.png: vSphere client|

For information on creating a port profile, see `Cisco Nexus 1000V Port
Profile Configuration
Guide <http://www.cisco.com/en/US/docs/switches/datacenter/nexus1000/sw/4_2_1_s_v_1_4_a/port_profile/configuration/guide/n1000v_port_profile.html>`__.

Assigning Physical NIC Adapters
'''''''''''''''''''''''''''''''

Assign ESXi host's physical NIC adapters, which correspond to each
physical network, to the port profiles. In each ESXi host that is part
of the vCenter cluster, observe the physical networks assigned to each
port profile and note down the names of the port profile for future use.
This mapping information helps you when configuring physical networks
during the zone configuration on CloudStack. These Ethernet port profile
names are later specified as VMware Traffic Labels for different traffic
types when configuring physical networks during the zone configuration.
For more information on configuring physical networks, see
`Section 8.4.6, “Configuring a vSphere Cluster with Nexus 1000v Virtual
Switch” <#vmware-vsphere-cluster-config-nexus-vswitch>`__.

Adding VLAN Ranges
''''''''''''''''''

Determine the public VLAN, System VLAN, and Guest VLANs to be used by
the CloudStack. Ensure that you add them to the port profile database.
Corresponding to each physical network, add the VLAN range to port
profiles. In the VSM command prompt, run the switchport trunk allowed
vlan<range> command to add the VLAN ranges to the port profile.

For example:

.. code:: bash

    switchport trunk allowed vlan 1,140-147,196-203

In this example, the allowed VLANs added are 1, 140-147, and 196-203

You must also add all the public and private VLANs or VLAN ranges to the
switch. This range is the VLAN range you specify in your zone.

.. note:: Before you run the vlan command, ensure that the configuration mode is enabled in Nexus 1000v virtual switch.

For example:

If you want the VLAN 200 to be used on the switch, run the following
command:

.. code:: bash

    vlan 200

If you want the VLAN range 1350-1750 to be used on the switch, run the
following command:

.. code:: bash

    vlan 1350-1750

Refer to Cisco Nexus 1000V Command Reference of specific product
version.

Enabling Nexus Virtual Switch in CloudStack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To make a CloudStack deployment Nexus enabled, you must set the
vmware.use.nexus.vswitch parameter true by using the Global Settings
page in the CloudStack UI. Unless this parameter is set to "true" and
restart the management server, you cannot see any UI options specific to
Nexus virtual switch, and CloudStack ignores the Nexus virtual switch
specific parameters specified in the AddTrafficTypeCmd,
UpdateTrafficTypeCmd, and AddClusterCmd API calls.

Unless the CloudStack global parameter "vmware.use.nexus.vswitch" is set
to "true", CloudStack by default uses VMware standard vSwitch for
virtual network infrastructure. In this release, CloudStack doesn’t
support configuring virtual networks in a deployment with a mix of
standard vSwitch and Nexus 1000v virtual switch. The deployment can have
either standard vSwitch or Nexus 1000v virtual switch.

Configuring Nexus 1000v Virtual Switch in CloudStack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can configure Nexus dvSwitch by adding the necessary resources while
the zone is being created.

|vmwarenexusaddcluster.png: vmware nexus add cluster|

After the zone is created, if you want to create an additional cluster
along with Nexus 1000v virtual switch in the existing zone, use the Add
Cluster option. For information on creating a cluster, see
`Section 6.5.2, “Add Cluster: vSphere” <#add-clusters-vsphere>`__.

In both these cases, you must specify the following parameters to
configure Nexus virtual switch:

=========================  =======================================================================================================================
Parameters                 Description
=========================  =======================================================================================================================
Cluster Name               Enter the name of the cluster you created in vCenter. For example,"cloud.cluster".
vCenter Host               Enter the host name or the IP address of the vCenter host where you have deployed the Nexus virtual switch.
vCenter User name          Enter the username that CloudStack should use to connect to vCenter. This user must have all administrative privileges.
vCenter Password           Enter the password for the user named above.
vCenter Datacenter         Enter the vCenter datacenter that the cluster is in. For example, "cloud.dc.VM".
Nexus dvSwitch IP Address  The IP address of the VSM component of the Nexus 1000v virtual switch.
Nexus dvSwitch Username    The admin name to connect to the VSM appliance.
Nexus dvSwitch Password    The corresponding password for the admin user specified above.
=========================  =======================================================================================================================


Removing Nexus Virtual Switch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. 

   In the vCenter datacenter that is served by the Nexus virtual switch,
   ensure that you delete all the hosts in the corresponding cluster.

#. 

   Log in with Admin permissions to the CloudStack administrator UI.

#. 

   In the left navigation bar, select Infrastructure.

#. 

   In the Infrastructure page, click View all under Clusters.

#. 

   Select the cluster where you want to remove the virtual switch.

#. 

   In the dvSwitch tab, click the name of the virtual switch.

#. 

   In the Details page, click Delete Nexus dvSwitch icon.
   |DeleteButton.png: button to delete dvSwitch|

   Click Yes in the confirmation dialog box.

Configuring a VMware Datacenter with VMware Distributed Virtual Switch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack supports VMware vNetwork Distributed Switch (VDS) for virtual
network configuration in a VMware vSphere environment. This section
helps you configure VMware VDS in a CloudStack deployment. Each vCenter
server instance can support up to 128 VDS instances and each VDS
instance can manage up to 500 VMware hosts.

About VMware Distributed Virtual Switch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

VMware VDS is an aggregation of host-level virtual switches on a VMware
vCenter server. VDS abstracts the configuration of individual virtual
switches that span across a large number of hosts, and enables
centralized provisioning, administration, and monitoring for your entire
datacenter from a centralized interface. In effect, a VDS acts as a
single virtual switch at the datacenter level and manages networking for
a number of hosts in a datacenter from a centralized VMware vCenter
server. Each VDS maintains network runtime state for VMs as they move
across multiple hosts, enabling inline monitoring and centralized
firewall services. A VDS can be deployed with or without Virtual
Standard Switch and a Nexus 1000V virtual switch.

Prerequisites and Guidelines
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  

   VMware VDS is supported only on Public and Guest traffic in
   CloudStack.

-  

   VMware VDS does not support multiple VDS per traffic type. If a user
   has many VDS switches, only one can be used for Guest traffic and
   another one for Public traffic.

-  

   Additional switches of any type can be added for each cluster in the
   same zone. While adding the clusters with different switch type,
   traffic labels is overridden at the cluster level.

-  

   Management and Storage network does not support VDS. Therefore, use
   Standard Switch for these networks.

-  

   When you remove a guest network, the corresponding dvportgroup will
   not be removed on the vCenter. You must manually delete them on the
   vCenter.

Preparation Checklist
^^^^^^^^^^^^^^^^^^^^^

For a smoother configuration of VMware VDS, note down the VDS name you
have added in the datacenter before you start:

|vds-name.png: Name of the dvSwitch as specified in the vCenter.|

Use this VDS name in the following:

-  

   The switch name in the Edit traffic label dialog while configuring a
   public and guest traffic during zone creation.

   During a zone creation, ensure that you select VMware vNetwork
   Distributed Virtual Switch when you configure guest and public
   traffic type.

   |traffic-type.png: virtual switch type|

-  

   The Public Traffic vSwitch Type field when you add a VMware
   VDS-enabled cluster.

-  

   The switch name in the traffic label while updating the switch type
   in a zone.

Traffic label format in the last case is [["Name of
vSwitch/dvSwitch/EthernetPortProfile"][,"VLAN ID"[,"vSwitch Type"]]]

The possible values for traffic labels are:

-  

   empty string

-  

   dvSwitch0

-  

   dvSwitch0,200

-  

   dvSwitch1,300,vmwaredvs

-  

   myEthernetPortProfile,,nexusdvs

-  

   dvSwitch0,,vmwaredvs


The three fields to fill in are:

- 

   Name of the virtual / distributed virtual switch at vCenter.

   The default value depends on the type of virtual switch:

   **vSwitch0**: If type of virtual switch is VMware vNetwork Standard virtual switch

   **dvSwitch0**: If type of virtual switch is VMware vNetwork Distributed virtual switch

   **epp0**: If type of virtual switch is Cisco Nexus 1000v Distributed virtual switch

-

   VLAN ID to be used for this traffic wherever applicable.

   This field would be used for only public traffic as of now. In case of guest traffic this field would be ignored and could be left empty for guest traffic. By default empty string would be assumed which translates to untagged VLAN for that specific traffic type.

-

   Type of virtual switch. Specified as string.

   Possible valid values are vmwaredvs, vmwaresvs, nexusdvs.

   **vmwaresvs**: Represents VMware vNetwork Standard virtual switch

   **vmwaredvs**: Represents VMware vNetwork distributed virtual switch

   **nexusdvs**: Represents Cisco Nexus 1000v distributed virtual switch.

   If nothing specified (left empty), zone-level default virtual switchwould be defaulted, based on the value of global parameter you specify.

   Following are the global configuration parameters:

   **vmware.use.dvswitch**: Set to true to enable any kind (VMware DVS and Cisco Nexus 1000v) of distributed virtual switch in a CloudStack deployment. If set to false, the virtual switch that can be used in that CloudStack deployment is Standard virtual switch.

   **vmware.use.nexus.vswitch**: This parameter is ignored if vmware.use.dvswitch is set to false. Set to true to enable Cisco Nexus 1000v distributed virtual switch in a CloudStack deployment.

Enabling Virtual Distributed Switch in CloudStack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To make a CloudStack deployment VDS enabled, set the vmware.use.dvswitch
parameter to true by using the Global Settings page in the CloudStack UI
and restart the Management Server. Unless you enable the
vmware.use.dvswitch parameter, you cannot see any UI options specific to
VDS, and CloudStack ignores the VDS-specific parameters that you
specify. Additionally, CloudStack uses VDS for virtual network
infrastructure if the value of vmware.use.dvswitch parameter is true and
the value of vmware.use.nexus.dvswitch parameter is false. Another
global parameter that defines VDS configuration is
vmware.ports.per.dvportgroup. This is the default number of ports per
VMware dvPortGroup in a VMware environment. Default value is 256. This
number directly associated with the number of guest network you can
create.

CloudStack supports orchestration of virtual networks in a deployment
with a mix of Virtual Distributed Switch, Standard Virtual Switch and
Nexus 1000v Virtual Switch.

Configuring Distributed Virtual Switch in CloudStack
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can configure VDS by adding the necessary resources while a zone is
created.

Alternatively, at the cluster level, you can create an additional
cluster with VDS enabled in the existing zone. Use the Add Cluster
option. For information as given in `Section 6.5.2, “Add Cluster:
vSphere” <#add-clusters-vsphere>`__.

In both these cases, you must specify the following parameters to
configure VDS:

|dvSwitchConfig.png: Configuring dvSwitch|

=================================   ===================================================================================================================
Parameters Description
=================================   ===================================================================================================================
Cluster Name                        Enter the name of the cluster you created in vCenter. For example, "cloudcluster".
vCenter Host                        Enter the name or the IP address of the vCenter host where you have deployed the VMware VDS.
vCenter User name                   Enter the username that CloudStack should use to connect to vCenter. This user must have all administrative privileges.
vCenter Password                    Enter the password for the user named above.
vCenter Datacenter                  Enter the vCenter datacenter that the cluster is in. For example, "clouddcVM".
Override Public Traffic             Enable this option to override the zone-wide public traffic for the cluster you are creating.
Public Traffic vSwitch Type         This option is displayed only if you enable the Override Public Traffic option. Select VMware vNetwork Distributed Virtual Switch. If the vmware.use.dvswitch global parameter is true, the default option will be VMware vNetwork Distributed Virtual Switch.
Public Traffic vSwitch Name         Name of virtual switch to be used for the public traffic.
Override Guest Traffic              Enable the option to override the zone-wide guest traffic for the cluster you are creating.
Guest Traffic vSwitch Type          This option is displayed only if you enable the Override Guest Traffic option. Select VMware vNetwork Distributed Virtual Switch. If the vmware.use.dvswitch global parameter is true, the default option will be VMware vNetwork Distributed Virtual Switch.
Guest Traffic vSwitch Name          Name of virtual switch to be used for guest traffic.
=================================   ===================================================================================================================


Storage Preparation for vSphere (iSCSI only)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use of iSCSI requires preparatory work in vCenter. You must add an iSCSI
target and create an iSCSI datastore.

If you are using NFS, skip this section.

Enable iSCSI initiator for ESXi hosts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. 

   In vCenter, go to hosts and Clusters/Configuration, and click Storage
   Adapters link. You will see:

   |vmwareiscsiinitiator.png: iscsi initiator|

#. 

   Select iSCSI software adapter and click Properties.

   |vmwareiscsiinitiatorproperties.png: iscsi initiator properties|

#. 

   Click the Configure... button.

   |vmwareiscsigeneral.png: iscsi general|

#. 

   Check Enabled to enable the initiator.

#. 

   Click OK to save.

Add iSCSI target
^^^^^^^^^^^^^^^^

Under the properties dialog, add the iSCSI target info:

|vmwareiscsitargetadd.png: iscsi target add|
   
Repeat these steps for all ESXi hosts in the cluster.

Create an iSCSI datastore
^^^^^^^^^^^^^^^^^^^^^^^^^

You should now create a VMFS datastore. Follow these steps to do so:

#. 

   Select Home/Inventory/Datastores.

#. 

   Right click on the datacenter node.

#. 

   Choose Add Datastore... command.

#. 

   Follow the wizard to create a iSCSI datastore.

This procedure should be done on one host in the cluster. It is not
necessary to do this on all hosts.

|vmwareiscsidatastore.png: iscsi datastore|

Multipathing for vSphere (Optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Storage multipathing on vSphere nodes may be done according to the
vSphere installation guide.

Add Hosts or Configure Clusters (vSphere)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use vCenter to create a vCenter cluster and add your desired hosts to
the cluster. You will later add the entire cluster to CloudStack. (see
`Section 6.5.2, “Add Cluster: vSphere” <#add-clusters-vsphere>`__).

Applying Hotfixes to a VMware vSphere Host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 

   Disconnect the VMware vSphere cluster from CloudStack. It should
   remain disconnected long enough to apply the hotfix on the host.

   #. 

      Log in to the CloudStack UI as root.

      See `Section 5.1, “Log In to the UI” <#log-in>`__.

   #. 

      Navigate to the VMware cluster, click Actions, and select
      Unmanage.

   #. 

      Watch the cluster status until it shows Unmanaged.

#. 

   Perform the following on each of the ESXi hosts in the cluster:

   #. 

      Move each of the ESXi hosts in the cluster to maintenance mode.

   #. 

      Ensure that all the VMs are migrated to other hosts in that
      cluster.

   #. 

      If there is only one host in that cluster, shutdown all the VMs
      and move the host into maintenance mode.

   #. 

      Apply the patch on the ESXi host.

   #. 

      Restart the host if prompted.

   #. 

      Cancel the maintenance mode on the host.

#. 

   Reconnect the cluster to CloudStack:

   #. 

      Log in to the CloudStack UI as root.

   #. 

      Navigate to the VMware cluster, click Actions, and select Manage.

   #. 

      Watch the status to see that all the hosts come up. It might take
      several minutes for the hosts to come up.

      Alternatively, verify the host state is properly synchronized and
      updated in the CloudStack database.

LXC Installation and Configuration
----------------------------------

System Requirements for LXC Hosts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LXC requires the Linux kernel cgroups functionality which is available
starting 2.6.24. Although you are not required to run these
distributions, the following are recommended:

-  

   CentOS / RHEL: 6.3

-  

   Ubuntu: 12.04(.1)

The main requirement for LXC hypervisors is the libvirt and Qemu
version. No matter what Linux distribution you are using, make sure the
following requirements are met:

-  

   libvirt: 1.0.0 or higher

-  

   Qemu/KVM: 1.0 or higher

The default bridge in CloudStack is the Linux native bridge
implementation (bridge module). CloudStack includes an option to work
with OpenVswitch, the requirements are listed below

-  

   libvirt: 1.0.0 or higher

-  

   openvswitch: 1.7.1 or higher

In addition, the following hardware requirements apply:

-  

   Within a single cluster, the hosts must be of the same distribution
   version.

-  

   All hosts within a cluster must be homogenous. The CPUs must be of
   the same type, count, and feature flags.

-  

   Must support HVM (Intel-VT or AMD-V enabled)

-  

   64-bit x86 CPU (more cores results in better performance)

-  

   4 GB of memory

-  

   At least 1 NIC

-  

   When you deploy CloudStack, the hypervisor host must not have any VMs
   already running

LXC Installation Overview
~~~~~~~~~~~~~~~~~~~~~~~~~

LXC does not have any native system VMs, instead KVM will be used to run
system VMs. This means that your host will need to support both LXC and
KVM, thus most of the installation and configuration will be identical
to the KVM installation. The material in this section doesn't duplicate
KVM installation docs. It provides the CloudStack-specific steps that
are needed to prepare a KVM host to work with CloudStack.

.. warning:: Before continuing, make sure that you have applied the latest updates to your host.

.. warning:: It is NOT recommended to run services on this host not controlled by CloudStack.

The procedure for installing an LXC Host is:

#. 

   Prepare the Operating System

#. 

   Install and configure libvirt

#. 

   Configure Security Policies (AppArmor and SELinux)

#. 

   Install and configure the Agent

Prepare the Operating System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OS of the Host must be prepared to host the CloudStack Agent and run
KVM instances.

#. 

   Log in to your OS as root.

#. 

   Check for a fully qualified hostname.

   .. code:: bash

       $ hostname --fqdn

   This should return a fully qualified hostname such as
   "kvm1.lab.example.org". If it does not, edit /etc/hosts so that it
   does.

#. 

   Make sure that the machine can reach the Internet.

   .. code:: bash

       $ ping www.cloudstack.org

#. 

   Turn on NTP for time synchronization.

   .. note:: NTP is required to synchronize the clocks of the servers in your
   cloud. Unsynchronized clocks can cause unexpected problems.

   #. 

      Install NTP

      .. code:: bash

          $ yum install ntp

      .. code:: bash

          $ apt-get install openntpd

#. 

   Repeat all of these steps on every hypervisor host.

Install and configure the Agent
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To manage LXC instances on the host CloudStack uses a Agent. This Agent
communicates with the Management server and controls all the instances
on the host.

First we start by installing the agent:

In RHEL or CentOS:

.. code:: bash

    $ yum install cloudstack-agent

In Ubuntu:

.. code:: bash

    $ apt-get install cloudstack-agent

Next step is to update the Agent configuration setttings. The settings
are in ``/etc/cloudstack/agent/agent.properties``

#. 

   Set the Agent to run in LXC mode:

   .. code:: bash

       hypervisor.type=lxc

#. 

   Optional: If you would like to use direct networking (instead of the
   default bridge networking), configure these lines:

   .. code:: bash

       libvirt.vif.driver=com.cloud.hypervisor.kvm.resource.DirectVifDriver

   .. code:: bash

       network.direct.source.mode=private

   .. code:: bash

       network.direct.device=eth0

The host is now ready to be added to a cluster. This is covered in a
later section, see `Section 6.6, “Adding a Host” <#host-add>`__. It is
recommended that you continue to read the documentation before adding
the host!

Install and Configure libvirt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack uses libvirt for managing virtual machines. Therefore it is
vital that libvirt is configured correctly. Libvirt is a dependency of
cloudstack-agent and should already be installed.

#. 

   In order to have live migration working libvirt has to listen for
   unsecured TCP connections. We also need to turn off libvirts attempt
   to use Multicast DNS advertising. Both of these settings are in
   ``/etc/libvirt/libvirtd.conf``

   Set the following parameters:

   .. code:: bash

       listen_tls = 0

   .. code:: bash

       listen_tcp = 1

   .. code:: bash

       tcp_port = "16509"

   .. code:: bash

       auth_tcp = "none"

   .. code:: bash

       mdns_adv = 0

#. 

   Turning on "listen\_tcp" in libvirtd.conf is not enough, we have to
   change the parameters as well:

   On RHEL or CentOS modify ``/etc/sysconfig/libvirtd``:

   Uncomment the following line:

   .. code:: bash

       #LIBVIRTD_ARGS="--listen"

   On Ubuntu: modify ``/etc/default/libvirt-bin``

   Add "-l" to the following line

   .. code:: bash

       libvirtd_opts="-d"

   so it looks like:

   .. code:: bash

       libvirtd_opts="-d -l"

#. 

   In order to have the VNC Console work we have to make sure it will
   bind on 0.0.0.0. We do this by editing ``/etc/libvirt/qemu.conf``

   Make sure this parameter is set:

   .. code:: bash

       vnc_listen = "0.0.0.0"

#. 

   Restart libvirt

   In RHEL or CentOS:

   .. code:: bash

       $ service libvirtd restart

   In Ubuntu:

   .. code:: bash

       $ service libvirt-bin restart

Configure the Security Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack does various things which can be blocked by security
mechanisms like AppArmor and SELinux. These have to be disabled to
ensure the Agent has all the required permissions.

#. 

   Configure SELinux (RHEL and CentOS)

   #. 

      Check to see whether SELinux is installed on your machine. If not,
      you can skip this section.

      In RHEL or CentOS, SELinux is installed and enabled by default.
      You can verify this with:

      .. code:: bash

          $ rpm -qa | grep selinux

   #. 

      Set the SELINUX variable in ``/etc/selinux/config`` to
      "permissive". This ensures that the permissive setting will be
      maintained after a system reboot.

      In RHEL or CentOS:

      .. code:: bash

          vi /etc/selinux/config

      Change the following line

      .. code:: bash

          SELINUX=enforcing

      to this

      .. code:: bash

          SELINUX=permissive

   #. 

      Then set SELinux to permissive starting immediately, without
      requiring a system reboot.

      .. code:: bash

          $ setenforce permissive

#. 

   Configure Apparmor (Ubuntu)

   #. 

      Check to see whether AppArmor is installed on your machine. If
      not, you can skip this section.

      In Ubuntu AppArmor is installed and enabled by default. You can
      verify this with:

      .. code:: bash

          $ dpkg --list 'apparmor'

   #. 

      Disable the AppArmor profiles for libvirt

      .. code::

          $ ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/

      .. code::

          $ ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/

      .. code::

          $ apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd

      .. code::

          $ apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper

Configure the network bridges
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning:: This is a very important section, please make sure you read this thoroughly.

.. note:: This section details how to configure bridges using the native implementation in Linux. Please refer to the next section if you intend to use OpenVswitch

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

#. 

   VLAN 100 for management of the hypervisor

#. 

   VLAN 200 for public network of the instances (cloudbr0)

#. 

   VLAN 300 for private network of the instances (cloudbr1)

On VLAN 100 we give the Hypervisor the IP-Address 192.168.42.11/24 with
the gateway 192.168.42.1

.. note:: The Hypervisor and Management server don't have to be in the same subnet!

Configuring the network bridges
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It depends on the distribution you are using how to configure these,
below you'll find examples for RHEL/CentOS and Ubuntu.

.. note:: The goal is to have two bridges called 'cloudbr0' and 'cloudbr1' after this section. This should be used as a guideline only. The exact configuration will depend on your network layout.

Configure in RHEL or CentOS
'''''''''''''''''''''''''''

The required packages were installed when libvirt was installed, we can
proceed to configuring the network.

First we configure eth0

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0

Make sure it looks similar to:

.. code:: bash

    DEVICE=eth0
    HWADDR=00:04:xx:xx:xx:xx
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    TYPE=Ethernet

We now have to configure the three VLAN interfaces:

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0.100

.. code:: bash

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

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0.200

.. code:: bash

    DEVICE=eth0.200
    HWADDR=00:04:xx:xx:xx:xx
    ONBOOT=yes
    HOTPLUG=no
    BOOTPROTO=none
    TYPE=Ethernet
    VLAN=yes
    BRIDGE=cloudbr0

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-eth0.300

.. code:: bash

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

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr0

Now we just configure it is a plain bridge without an IP-Address

.. code:: bash

    DEVICE=cloudbr0
    TYPE=Bridge
    ONBOOT=yes
    BOOTPROTO=none
    IPV6INIT=no
    IPV6_AUTOCONF=no
    DELAY=5
    STP=yes

We do the same for cloudbr1

.. code:: bash

    vi /etc/sysconfig/network-scripts/ifcfg-cloudbr1

.. code:: bash

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

.. warning:: Make sure you have an alternative way like IPMI or ILO to reach the machine in case you made a configuration error and the network stops functioning!

Configure in Ubuntu
'''''''''''''''''''

All the required packages were installed when you installed libvirt, so
we only have to configure the network.

.. code:: bash

    vi /etc/network/interfaces

Modify the interfaces file to look like this:

.. code:: bash

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

.. warning:: Make sure you have an alternative way like IPMI or ILO to reach the machine in case you made a configuration error and the network stops functioning!

Configuring the firewall
~~~~~~~~~~~~~~~~~~~~~~~~

The hypervisor needs to be able to communicate with other hypervisors
and the management server needs to be able to reach the hypervisor.

In order to do so we have to open the following TCP ports (if you are
using a firewall):

#. 

   22 (SSH)

#. 

   1798

#. 

   16509 (libvirt)

#. 

   5900 - 6100 (VNC consoles)

#. 

   49152 - 49216 (libvirt live migration)

It depends on the firewall you are using how to open these ports. Below
you'll find examples how to open these ports in RHEL/CentOS and Ubuntu.

Open ports in RHEL/CentOS
^^^^^^^^^^^^^^^^^^^^^^^^^

RHEL and CentOS use iptables for firewalling the system, you can open
extra ports by executing the following iptable commands:

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 22 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 1798 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 16509 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 5900:6100 -j ACCEPT

.. code:: bash

    $ iptables -I INPUT -p tcp -m tcp --dport 49152:49216 -j ACCEPT

These iptable settings are not persistent accross reboots, we have to
save them first.

.. code:: bash

    $ iptables-save > /etc/sysconfig/iptables

Open ports in Ubuntu
^^^^^^^^^^^^^^^^^^^^

The default firewall under Ubuntu is UFW (Uncomplicated FireWall), which
is a Python wrapper around iptables.

To open the required ports, execute the following commands:

.. code:: bash

    $ ufw allow proto tcp from any to any port 22

.. code:: bash

    $ ufw allow proto tcp from any to any port 1798

.. code:: bash

    $ ufw allow proto tcp from any to any port 16509

.. code:: bash

    $ ufw allow proto tcp from any to any port 5900:6100

.. code:: bash

    $ ufw allow proto tcp from any to any port 49152:49216

.. note:: By default UFW is not enabled on Ubuntu. Executing these commands with
the firewall disabled does not enable the firewall.

Add the host to CloudStack
~~~~~~~~~~~~~~~~~~~~~~~~~~

The host is now ready to be added to a cluster. This is covered in a
later section, see `Section 6.6, “Adding a Host” <#host-add>`__. It is
recommended that you continue to read the documentation before adding
the host!



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
.. |vspherephysicalnetwork.png: vSphere client| image:: ./_static/images/vmware-physical-network.png
.. |vsphereincreaseports.png: vSphere client| image:: ./_static/images/vmware-increase-ports.png
.. |vspherevswitchproperties.png: vSphere client| image:: ./_static/images/vmware-vswitch-properties.png
.. |vspheremgtnetwork.png: vSphere client| image:: ./_static/images/vmware-mgt-network-properties.png
.. |vmwarenexusportprofile.png: vSphere client| image:: ./_static/images/vmware-nexus-port-profile.png
.. |vmwarenexusaddcluster.png: vmware nexus add cluster| image:: ./_static/images/vmware-nexus-add-cluster.png
.. |vmwareiscsiinitiator.png: iscsi initiator| image:: ./_static/images/vmware-iscsi-initiator.png
.. |vmwareiscsiinitiatorproperties.png: iscsi initiator properties| image:: ./_static/images/vmware-iscsi-initiator-properties.png
.. |vmwareiscsigeneral.png: iscsi general| image:: ./_static/images/vmware-iscsi-general.png
.. |vmwareiscsitargetadd.png: iscsi target add| image:: ./_static/images/vmware-iscsi-target-add.png
.. |vmwareiscsidatastore.png: iscsi datastore| image:: ./_static/images/vmware-iscsi-datastore.png
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
