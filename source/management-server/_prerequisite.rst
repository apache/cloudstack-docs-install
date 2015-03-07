Management Server Installation Overview
---------------------------------------

This section describes installing the Management Server. There are two
slightly different installation flows, depending on how many Management
Server nodes will be in your cloud:

-  A single Management Server node, with MySQL on the same node.

-  Multiple Management Server nodes, with MySQL on a node separate from
   the Management Servers.

In either case, each machine must meet the system requirements described
in :ref:`minimum-system-requirements`.

.. warning::
   For the sake of security, be sure the public Internet can not access port 
   8096 or port 8250 on the Management Server.

The procedure for installing the Management Server is:

#. Prepare the Operating System

#. (XenServer only) Download and install vhd-util.

#. Install the First Management Server

#. Install and Configure the MySQL database

#. Prepare NFS Shares

#. Prepare and Start Additional Management Servers (optional)

#. Prepare the System VM Template


Prepare the Operating System
----------------------------

The OS must be prepared to host the Management Server using the
following steps. These steps must be performed on each Management Server
node.

#. Log in to your OS as root.

#. Check for a fully qualified hostname.

   .. sourcecode:: bash

      hostname --fqdn

   This should return a fully qualified hostname such as
   "management1.lab.example.org". If it does not, edit /etc/hosts so
   that it does.

#. Make sure that the machine can reach the Internet.

   .. sourcecode:: bash

      ping www.cloudstack.org

#. Turn on NTP for time synchronization.

   .. note::
      NTP is required to synchronize the clocks of the servers in your cloud.

   Install NTP.

   .. sourcecode:: bash

      yum install ntp

   .. sourcecode:: bash

      sudo apt-get install openntpd

#. Repeat all of these steps on every host where the Management Server
   will be installed.

