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

Installation
============

Who Should Read This
--------------------

For those who have already gone through a design phase and planned a
more sophisticated deployment, or those who are ready to start scaling
up a trial installation. With the following procedures, you can start
using the more powerful features of CloudStack, such as advanced VLAN
networking, high availability, additional network elements such as load
balancers and firewalls, and support for multiple hypervisors including
Citrix XenServer, KVM, and VMware vSphere.

Overview of Installation Steps
------------------------------

For anything more than a simple trial installation, you will need
guidance for a variety of configuration choices. It is strongly
recommended that you read the following:

-  

   Choosing a Deployment Architecture

-  

   Choosing a Hypervisor: Supported Features

-  

   Network Setup

-  

   Storage Setup

-  

   Best Practices

#. 

   Make sure you have the required hardware ready. See `Section 4.3,
   “Minimum System Requirements” <#minimum-system-requirements>`__

#. 

   Install the Management Server (choose single-node or multi-node). See
   `Section 4.5, “Management Server
   Installation” <#management-server-install-flow>`__

#. 

   Log in to the UI. See `Chapter 5, *User Interface* <#ui>`__

#. 

   Add a zone. Includes the first pod, cluster, and host. See
   `Section 6.3, “Adding a Zone” <#zone-add>`__

#. 

   Add more pods (optional). See `Section 6.4, “Adding a
   Pod” <#pod-add>`__

#. 

   Add more clusters (optional). See `Section 6.5, “Adding a
   Cluster” <#cluster-add>`__

#. 

   Add more hosts (optional). See `Section 6.6, “Adding a
   Host” <#host-add>`__

#. 

   Add more primary storage (optional). See `Section 6.7, “Add Primary
   Storage” <#primary-storage-add>`__

#. 

   Add more secondary storage (optional). See `Section 6.8, “Add
   Secondary Storage” <#secondary-storage-add>`__

#. 

   Try using the cloud. See `Section 6.9, “Initialize and
   Test” <#initialize-and-test>`__

Minimum System Requirements
---------------------------

Management Server, Database, and Storage System Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The machines that will run the Management Server and MySQL database must
meet the following requirements. The same machines can also be used to
provide primary and secondary storage, such as via localdisk or NFS. The
Management Server may be placed on a virtual machine.

-  

   Operating system:

   -  

      Preferred: CentOS/RHEL 6.3+ or Ubuntu 12.04(.1)

-  

   64-bit x86 CPU (more cores results in better performance)

-  

   4 GB of memory

-  

   250 GB of local disk (more results in better capability; 500 GB
   recommended)

-  

   At least 1 NIC

-  

   Statically allocated IP address

-  

   Fully qualified domain name as returned by the hostname command

Host/Hypervisor System Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The host is where the cloud services run in the form of guest virtual
machines. Each host is one machine that meets the following
requirements:

-  

   Must support HVM (Intel-VT or AMD-V enabled).

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

.. note::If DHCP is used for hosts, ensure that no conflict occurs between DHCP server used for these hosts and the DHCP router created by CloudStack.

-  

   Latest hotfixes applied to hypervisor software

-  

   When you deploy CloudStack, the hypervisor host must not have any VMs
   already running

-  

   All hosts within a cluster must be homogeneous. The CPUs must be of
   the same type, count, and feature flags.

Hosts have additional requirements depending on the hypervisor. See the
requirements listed at the top of the Installation section for your
chosen hypervisor:

.. warning::Be sure you fulfill the additional hypervisor requirements and installation steps provided in this Guide. Hypervisor hosts must be properly prepared to work with CloudStack. For example, the requirements for XenServer are listed under Citrix XenServer Installation.

Configure package repository
----------------------------

CloudStack is only distributed from source from the official mirrors.
However, members of the CloudStack community may build convenience
binaries so that users can install Apache CloudStack without needing to
build from source.

If you didn't follow the steps to build your own packages from source in
the sections for `Section 3.6, “Building RPMs from
Source” <#sect-source-buildrpm>`__ or `Section 3.5, “Building DEB
packages” <#sect-source-builddebs>`__ you may find pre-built DEB and RPM
packages for your convenience linked from the
`downloads <http://cloudstack.apache.org/downloads.html>`__ page.

.. note::These repositories contain both the Management Server and KVM Hypervisor packages.

DEB package repository
~~~~~~~~~~~~~~~~~~~~~~

You can add a DEB package repository to your apt sources with the
following commands. Please note that only packages for Ubuntu 12.04 LTS
(precise) are being built at this time.

Use your preferred editor and open (or create)
``/etc/apt/sources.list.d/cloudstack.list``. Add the community provided
repository to the file:

.. code:: bash

    deb http://cloudstack.apt-get.eu/ubuntu precise 4.2

We now have to add the public key to the trusted keys.

.. code:: bash

    $ wget -O - http://cloudstack.apt-get.eu/release.asc|apt-key add -

Now update your local apt cache.

.. code:: bash

    $ apt-get update

Your DEB package repository should now be configured and ready for use.

RPM package repository
~~~~~~~~~~~~~~~~~~~~~~

There is a RPM package repository for CloudStack so you can easily
install on RHEL based platforms.

If you're using an RPM-based system, you'll want to add the Yum
repository so that you can install CloudStack with Yum.

Yum repository information is found under ``/etc/yum.repos.d``. You'll
see several ``.repo`` files in this directory, each one denoting a
specific repository.

To add the CloudStack repository, create
``/etc/yum.repos.d/cloudstack.repo`` and insert the following
information.

.. code:: bash

    [cloudstack]
    name=cloudstack
    baseurl=http://cloudstack.apt-get.eu/rhel/4.2/
    enabled=1
    gpgcheck=0

Now you should be able to install CloudStack using Yum.

Management Server Installation
------------------------------

Management Server Installation Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section describes installing the Management Server. There are two
slightly different installation flows, depending on how many Management
Server nodes will be in your cloud:

-  

   A single Management Server node, with MySQL on the same node.

-  

   Multiple Management Server nodes, with MySQL on a node separate from
   the Management Servers.

In either case, each machine must meet the system requirements described
in System Requirements.

.. warning::For the sake of security, be sure the public Internet can not access
port 8096 or port 8250 on the Management Server.

The procedure for installing the Management Server is:

#. 

   Prepare the Operating System

#. 

   (XenServer only) Download and install vhd-util.

#. 

   Install the First Management Server

#. 

   Install and Configure the MySQL database

#. 

   Prepare NFS Shares

#. 

   Prepare and Start Additional Management Servers (optional)

#. 

   Prepare the System VM Template

Prepare the Operating System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OS must be prepared to host the Management Server using the
following steps. These steps must be performed on each Management Server
node.

#. 

   Log in to your OS as root.

#. 

   Check for a fully qualified hostname.

   .. code:: bash

       hostname --fqdn

   This should return a fully qualified hostname such as
   "management1.lab.example.org". If it does not, edit /etc/hosts so
   that it does.

#. 

   Make sure that the machine can reach the Internet.

   .. code:: bash

       ping www.cloudstack.org

#. 

   Turn on NTP for time synchronization.

.. note::NTP is required to synchronize the clocks of the servers in your
   cloud.

   #. 

      Install NTP.

      .. code:: bash

          yum install ntp

      .. code:: bash

          apt-get install openntpd

#. 

   Repeat all of these steps on every host where the Management Server
   will be installed.

Install the Management Server on the First Host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first step in installation, whether you are installing the
Management Server on one host or many, is to install the software on a
single node.

.. note::If you are planning to install the Management Server on multiple nodes
for high availability, do not proceed to the additional nodes yet. That
step will come later.

The CloudStack Management server can be installed using either RPM or
DEB packages. These packages will depend on everything you need to run
the Management server.

Install on CentOS/RHEL
^^^^^^^^^^^^^^^^^^^^^^

We start by installing the required packages:

.. code:: bash

    yum install cloudstack-management

Install on Ubuntu
^^^^^^^^^^^^^^^^^

.. code:: bash

    apt-get install cloudstack-management

Downloading vhd-util
^^^^^^^^^^^^^^^^^^^^

This procedure is required only for installations where XenServer is
installed on the hypervisor hosts.

Before setting up the Management Server, download vhd-util from
`vhd-util <http://download.cloud.com.s3.amazonaws.com/tools/vhd-util>`__.

If the Management Server is RHEL or CentOS, copy vhd-util to
/usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver.

If the Management Server is Ubuntu, copy vhd-util to
/usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver.

Install the database server
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The CloudStack management server uses a MySQL database server to store
its data. When you are installing the management server on a single
node, you can install the MySQL server locally. For an installation that
has multiple management server nodes, we assume the MySQL database also
runs on a separate node.

CloudStack has been tested with MySQL 5.1 and 5.5. These versions are
included in RHEL/CentOS and Ubuntu.

Install the Database on the Management Server Node
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes how to install MySQL on the same machine with the
Management Server. This technique is intended for a simple deployment
that has a single Management Server node. If you have a multi-node
Management Server deployment, you will typically use a separate node for
MySQL. See `Section 4.5.4.2, “Install the Database on a Separate
Node” <#management-server-install-db-external>`__.

#. 

   Install MySQL from the package repository of your distribution:

   .. code:: bash

       yum install mysql-server

   .. code:: bash

       apt-get install mysql-server

#. 

   Open the MySQL configuration file. The configuration file is
   ``/etc/my.cnf`` or ``/etc/mysql/my.cnf``, depending on your OS.

#. 

   Insert the following lines in the [mysqld] section.

   You can put these lines below the datadir line. The max\_connections
   parameter should be set to 350 multiplied by the number of Management
   Servers you are deploying. This example assumes one Management
   Server.

.. note::On Ubuntu, you can also create a file
   `/etc/mysql/conf.d/cloudstack.cnf` and add these directives there.
   Don't forget to add [mysqld] on the first line of the file.

   .. code:: bash

       innodb_rollback_on_timeout=1
       innodb_lock_wait_timeout=600
       max_connections=350
       log-bin=mysql-bin
       binlog-format = 'ROW'

#. 

   Start or restart MySQL to put the new configuration into effect.

   On RHEL/CentOS, MySQL doesn't automatically start after installation.
   Start it manually.

   .. code:: bash

       service mysqld start

   On Ubuntu, restart MySQL.

   .. code:: bash

       service mysql restart

#. 

   (CentOS and RHEL only; not required on Ubuntu)

   Warning
   
   On RHEL and CentOS, MySQL does not set a root password by default. It
   is very strongly recommended that you set a root password as a
   security precaution.

   Run the following command to secure your installation. You can answer
   "Y" to all questions.

   .. code:: bash

       mysql_secure_installation

#. 

   CloudStack can be blocked by security mechanisms, such as SELinux.
   Disable SELinux to ensure + that the Agent has all the required
   permissions.

   Configure SELinux (RHEL and CentOS):

   #. 

      Check whether SELinux is installed on your machine. If not, you
      can skip this section.

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

      to this:

      .. code:: bash

          SELINUX=permissive

   #. 

      Set SELinux to permissive starting immediately, without requiring
      a system reboot.

      .. code:: bash

          $ setenforce permissive

#. 

   Set up the database. The following command creates the "cloud" user
   on the database.

   -  

      In dbpassword, specify the password to be assigned to the "cloud"
      user. You can choose to provide no password although that is not
      recommended.

   -  

      In deploy-as, specify the username and password of the user
      deploying the database. In the following command, it is assumed
      the root user is deploying the database and creating the "cloud"
      user.

   -  

      (Optional) For encryption\_type, use file or web to indicate the
      technique used to pass in the database encryption password.
      Default: file. See `Section 4.5.5, “About Password and Key
      Encryption” <#about-password-encryption>`__.

   -  

      (Optional) For management\_server\_key, substitute the default key
      that is used to encrypt confidential parameters in the CloudStack
      properties file. Default: password. It is highly recommended that
      you replace this with a more secure value. See `Section 4.5.5,
      “About Password and Key
      Encryption” <#about-password-encryption>`__.

   -  

      (Optional) For database\_key, substitute the default key that is
      used to encrypt confidential parameters in the CloudStack
      database. Default: password. It is highly recommended that you
      replace this with a more secure value. See `Section 4.5.5, “About
      Password and Key Encryption” <#about-password-encryption>`__.

   -  

      (Optional) For management\_server\_ip, you may explicitly specify
      cluster management server node IP. If not specified, the local IP
      address will be used.

   .. code:: bash

       cloudstack-setup-databases cloud:<dbpassword>@localhost \
       --deploy-as=root:<password> \
       -e <encryption_type> \
       -m <management_server_key> \
       -k <database_key> \
       -i <management_server_ip>

   When this script is finished, you should see a message like
   “Successfully initialized the database.”

   Note


   If the script is unable to connect to the MySQL database, check the
   "localhost" loopback address in ``/etc/hosts``. It should be pointing
   to the IPv4 loopback address "127.0.0.1" and not the IPv6 loopback
   address ::1. Alternatively, reconfigure MySQL to bind to the IPv6
   loopback interface.

#. 

   If you are running the KVM hypervisor on the same machine with the
   Management Server, edit /etc/sudoers and add the following line:

   .. code:: bash

       Defaults:cloud !requiretty

#. 

   Now that the database is set up, you can finish configuring the OS
   for the Management Server. This command will set up iptables,
   sudoers, and start the Management Server.

   .. code:: bash

       # cloudstack-setup-management

   You should see the message “CloudStack Management Server setup is
   done.”

Install the Database on a Separate Node
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes how to install MySQL on a standalone machine,
separate from the Management Server. This technique is intended for a
deployment that includes several Management Server nodes. If you have a
single-node Management Server deployment, you will typically use the
same node for MySQL. See `Section 4.5.4.1, “Install the Database on the
Management Server Node” <#management-server-install-db-local>`__.

.. note:: The management server doesn't require a specific distribution for the
MySQL node. You can use a distribution or Operating System of your
choice. Using the same distribution as the management server is
recommended, but not required. See `Section 4.3.1, “Management Server,
Database, and Storage System Requirements” <#management-server-system-requirements>`__.

#. 

   Install MySQL from the package repository from your distribution:

   .. code:: bash

       yum install mysql-server

   .. code:: bash

       apt-get install mysql-server

#. 

   Edit the MySQL configuration (/etc/my.cnf or /etc/mysql/my.cnf,
   depending on your OS) and insert the following lines in the [mysqld]
   section. You can put these lines below the datadir line. The
   max\_connections parameter should be set to 350 multiplied by the
   number of Management Servers you are deploying. This example assumes
   two Management Servers.

.. note:: On Ubuntu, you can also create /etc/mysql/conf.d/cloudstack.cnf file
   and add these directives there. Don't forget to add [mysqld] on the
   first line of the file.

.. code:: bash

       innodb_rollback_on_timeout=1
       innodb_lock_wait_timeout=600
       max_connections=700
       log-bin=mysql-bin
       binlog-format = 'ROW'
       bind-address = 0.0.0.0

#. 

   Start or restart MySQL to put the new configuration into effect.

   On RHEL/CentOS, MySQL doesn't automatically start after installation.
   Start it manually.

   .. code:: bash

       service mysqld start

   On Ubuntu, restart MySQL.

   .. code:: bash

       service mysql restart

#. 

   (CentOS and RHEL only; not required on Ubuntu)

.. warning:: On RHEL and CentOS, MySQL does not set a root password by default. It
   is very strongly recommended that you set a root password as a
   security precaution. Run the following command to secure your installation. You can answer
   "Y" to all questions except "Disallow root login remotely?". Remote
   root login is required to set up the databases.

   .. code:: bash

       mysql_secure_installation

#. 

   If a firewall is present on the system, open TCP port 3306 so
   external MySQL connections can be established.

   On Ubuntu, UFW is the default firewall. Open the port with this
   command:

   .. code:: bash

       ufw allow mysql

   On RHEL/CentOS:

   #. 

      Edit the /etc/sysconfig/iptables file and add the following line
      at the beginning of the INPUT chain.

      .. code:: bash

          -A INPUT -p tcp --dport 3306 -j ACCEPT

   #. 

      Now reload the iptables rules.

      .. code:: bash

          service iptables restart

#. 

   Return to the root shell on your first Management Server.

#. 

   Set up the database. The following command creates the cloud user on
   the database.

   -  

      In dbpassword, specify the password to be assigned to the cloud
      user. You can choose to provide no password.

   -  

      In deploy-as, specify the username and password of the user
      deploying the database. In the following command, it is assumed
      the root user is deploying the database and creating the cloud
      user.

   -  

      (Optional) For encryption\_type, use file or web to indicate the
      technique used to pass in the database encryption password.
      Default: file. See `Section 4.5.5, “About Password and Key
      Encryption” <#about-password-encryption>`__.

   -  

      (Optional) For management\_server\_key, substitute the default key
      that is used to encrypt confidential parameters in the CloudStack
      properties file. Default: password. It is highly recommended that
      you replace this with a more secure value. See About Password and
      Key Encryption.

   -  

      (Optional) For database\_key, substitute the default key that is
      used to encrypt confidential parameters in the CloudStack
      database. Default: password. It is highly recommended that you
      replace this with a more secure value. See `Section 4.5.5, “About
      Password and Key Encryption” <#about-password-encryption>`__.

   -  

      (Optional) For management\_server\_ip, you may explicitly specify
      cluster management server node IP. If not specified, the local IP
      address will be used.

   .. code:: bash

       cloudstack-setup-databases cloud:<dbpassword>@<ip address mysql server> \
       --deploy-as=root:<password> \
       -e <encryption_type> \
       -m <management_server_key> \
       -k <database_key> \
       -i <management_server_ip>

   When this script is finished, you should see a message like
   “Successfully initialized the database.”

About Password and Key Encryption
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CloudStack stores several sensitive passwords and secret keys that are
used to provide security. These values are always automatically
encrypted:

-  

   Database secret key

-  

   Database password

-  

   SSH keys

-  

   Compute node root password

-  

   VPN password

-  

   User API secret key

-  

   VNC password

CloudStack uses the Java Simplified Encryption (JASYPT) library. The
data values are encrypted and decrypted using a database secret key,
which is stored in one of CloudStack’s internal properties files along
with the database password. The other encrypted values listed above,
such as SSH keys, are in the CloudStack internal database.

Of course, the database secret key itself can not be stored in the open
– it must be encrypted. How then does CloudStack read it? A second
secret key must be provided from an external source during Management
Server startup. This key can be provided in one of two ways: loaded from
a file or provided by the CloudStack administrator. The CloudStack
database has a configuration setting that lets it know which of these
methods will be used. If the encryption type is set to "file," the key
must be in a file in a known location. If the encryption type is set to
"web," the administrator runs the utility
com.cloud.utils.crypt.EncryptionSecretKeySender, which relays the key to
the Management Server over a known port.

The encryption type, database secret key, and Management Server secret
key are set during CloudStack installation. They are all parameters to
the CloudStack database setup script (cloudstack-setup-databases). The
default values are file, password, and password. It is, of course,
highly recommended that you change these to more secure keys.

Changing the Default Password Encryption
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Passwords are encoded when creating or updating users. CloudStack allows
you to determine the default encoding and authentication mechanism for
admin and user logins. Two new configurable lists have been
introduced—userPasswordEncoders and userAuthenticators.
userPasswordEncoders allows you to configure the order of preference for
encoding passwords, whereas userAuthenticators allows you to configure
the order in which authentication schemes are invoked to validate user
passwords.

Additionally, the plain text user authenticator has been modified not to
convert supplied passwords to their md5 sums before checking them with
the database entries. It performs a simple string comparison between
retrieved and supplied login passwords instead of comparing the
retrieved md5 hash of the stored password against the supplied md5 hash
of the password because clients no longer hash the password. The
following method determines what encoding scheme is used to encode the
password supplied during user creation or modification.

When a new user is created, the user password is encoded by using the
first valid encoder loaded as per the sequence specified in the
``UserPasswordEncoders`` property in the ``ComponentContext.xml`` or
``nonossComponentContext.xml`` files. The order of authentication
schemes is determined by the ``UserAuthenticators`` property in the same
files. If Non-OSS components, such as VMware environments, are to be
deployed, modify the ``UserPasswordEncoders`` and ``UserAuthenticators``
lists in the ``nonossComponentContext.xml`` file, for OSS environments,
such as XenServer or KVM, modify the ``ComponentContext.xml`` file. It
is recommended to make uniform changes across both the files. When a new
authenticator or encoder is added, you can add them to this list. While
doing so, ensure that the new authenticator or encoder is specified as a
bean in both these files. The administrator can change the ordering of
both these properties as preferred to change the order of schemes.
Modify the following list properties available in
``client/tomcatconf/nonossComponentContext.xml.in`` or
``client/tomcatconf/componentContext.xml.in`` as applicable, to the
desired order:

.. code:: bash

    <property name="UserAuthenticators">
             <list>
                <ref bean="SHA256SaltedUserAuthenticator"/>
                <ref bean="MD5UserAuthenticator"/>
                <ref bean="LDAPUserAuthenticator"/>
                <ref bean="PlainTextUserAuthenticator"/>
            </list>
        </property>
        <property name="UserPasswordEncoders">
            <list>
                <ref bean="SHA256SaltedUserAuthenticator"/>
                 <ref bean="MD5UserAuthenticator"/>
                 <ref bean="LDAPUserAuthenticator"/>
                <ref bean="PlainTextUserAuthenticator"/>
                </list>

In the above default ordering, SHA256Salt is used first for
``UserPasswordEncoders``. If the module is found and encoding returns a
valid value, the encoded password is stored in the user table's password
column. If it fails for any reason, the MD5UserAuthenticator will be
tried next, and the order continues. For ``UserAuthenticators``,
SHA256Salt authentication is tried first. If it succeeds, the user is
logged into the Management server. If it fails, md5 is tried next, and
attempts continues until any of them succeeds and the user logs in . If
none of them works, the user is returned an invalid credential message.

Prepare NFS Shares
~~~~~~~~~~~~~~~~~~

CloudStack needs a place to keep primary and secondary storage (see
Cloud Infrastructure Overview). Both of these can be NFS shares. This
section tells how to set up the NFS shares before adding the storage to
CloudStack.

NFS is not the only option for primary or secondary storage. For
example, you may use Ceph RBD, GlusterFS, iSCSI, and others. The choice
of storage system will depend on the choice of hypervisor and whether
you are dealing with primary or secondary storage.

The requirements for primary and secondary storage are described in:

-  

   `Section 2.6, “About Primary Storage” <#about-primary-storage>`__

-  

   `Section 2.7, “About Secondary Storage” <#about-secondary-storage>`__

A production installation typically uses a separate NFS server. See
`Section 4.5.7.1, “Using a Separate NFS
Server” <#nfs-shares-on-separate-server>`__.

You can also use the Management Server node as the NFS server. This is
more typical of a trial installation, but is technically possible in a
larger deployment. See `Section 4.5.7.2, “Using the Management Server as
the NFS Server” <#nfs-shares-on-management-server>`__.

Using a Separate NFS Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section tells how to set up NFS shares for secondary and
(optionally) primary storage on an NFS server running on a separate node
from the Management Server.

The exact commands for the following steps may vary depending on your
operating system version.

Warning


(KVM only) Ensure that no volume is already mounted at your NFS mount
point.

#. 

   On the storage server, create an NFS share for secondary storage and,
   if you are using NFS for primary storage as well, create a second NFS
   share. For example:

   .. code:: bash

       # mkdir -p /export/primary
       # mkdir -p /export/secondary

#. 

   To configure the new directories as NFS exports, edit /etc/exports.
   Export the NFS share(s) with
   rw,async,no\_root\_squash,no\_subtree\_check. For example:

   .. code:: bash

       # vi /etc/exports

   Insert the following line.

   .. code:: bash

       /export  *(rw,async,no_root_squash,no_subtree_check)

#. 

   Export the /export directory.

   .. code:: bash

       # exportfs -a

#. 

   On the management server, create a mount point for secondary storage.
   For example:

   .. code:: bash

       # mkdir -p /mnt/secondary

#. 

   Mount the secondary storage on your Management Server. Replace the
   example NFS server name and NFS share paths below with your own.

   .. code:: bash

       # mount -t nfs nfsservername:/nfs/share/secondary /mnt/secondary

Using the Management Server as the NFS Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section tells how to set up NFS shares for primary and secondary
storage on the same node with the Management Server. This is more
typical of a trial installation, but is technically possible in a larger
deployment. It is assumed that you will have less than 16TB of storage
on the host.

The exact commands for the following steps may vary depending on your
operating system version.

#. 

   On RHEL/CentOS systems, you'll need to install the nfs-utils package:

   .. code:: bash

       $ sudo yum install nfs-utils

#. 

   On the Management Server host, create two directories that you will
   use for primary and secondary storage. For example:

   .. code:: bash

       # mkdir -p /export/primary
       # mkdir -p /export/secondary

#. 

   To configure the new directories as NFS exports, edit /etc/exports.
   Export the NFS share(s) with
   rw,async,no\_root\_squash,no\_subtree\_check. For example:

   .. code:: bash

       # vi /etc/exports

   Insert the following line.

   .. code:: bash

       /export  *(rw,async,no_root_squash,no_subtree_check)

#. 

   Export the /export directory.

   .. code:: bash

       # exportfs -a

#. 

   Edit the /etc/sysconfig/nfs file.

   .. code:: bash

       # vi /etc/sysconfig/nfs

   Uncomment the following lines:

   .. code:: bash

       LOCKD_TCPPORT=32803
       LOCKD_UDPPORT=32769
       MOUNTD_PORT=892
       RQUOTAD_PORT=875
       STATD_PORT=662
       STATD_OUTGOING_PORT=2020

#. 

   Edit the /etc/sysconfig/iptables file.

   .. code:: bash

       # vi /etc/sysconfig/iptables

   Add the following lines at the beginning of the INPUT chain, where
   <NETWORK> is the network that you'll be using:

   .. code:: bash

       -A INPUT -s <NETWORK> -m state --state NEW -p udp --dport 111 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p tcp --dport 111 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p tcp --dport 2049 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p tcp --dport 32803 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p udp --dport 32769 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p tcp --dport 892 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p udp --dport 892 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p tcp --dport 875 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p udp --dport 875 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p tcp --dport 662 -j ACCEPT
       -A INPUT -s <NETWORK> -m state --state NEW -p udp --dport 662 -j ACCEPT                

#. 

   Run the following commands:

   .. code:: bash

       # service iptables restart
       # service iptables save

#. 

   If NFS v4 communication is used between client and server, add your
   domain to /etc/idmapd.conf on both the hypervisor host and Management
   Server.

   .. code:: bash

       # vi /etc/idmapd.conf

   Remove the character # from the beginning of the Domain line in
   idmapd.conf and replace the value in the file with your own domain.
   In the example below, the domain is company.com.

   .. code:: bash

       Domain = company.com

#. 

   Reboot the Management Server host.

   Two NFS shares called /export/primary and /export/secondary are now
   set up.

#. 

   It is recommended that you test to be sure the previous steps have
   been successful.

   #. 

      Log in to the hypervisor host.

   #. 

      Be sure NFS and rpcbind are running. The commands might be
      different depending on your OS. For example:

      .. code:: bash

          # service rpcbind start
          # service nfs start
          # chkconfig nfs on
          # chkconfig rpcbind on
          # reboot

   #. 

      Log back in to the hypervisor host and try to mount the /export
      directories. For example, substitute your own management server
      name:

      .. code:: bash

          # mkdir /primary
          # mount -t nfs <management-server-name>:/export/primary
          # umount /primary
          # mkdir /secondary
          # mount -t nfs <management-server-name>:/export/secondary
          # umount /secondary

Prepare and Start Additional Management Servers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For your second and subsequent Management Servers, you will install the
Management Server software, connect it to the database, and set up the
OS for the Management Server.

#. 

   Perform the steps in `Section 4.5.2, “Prepare the Operating
   System” <#prepare-os>`__ and `Section 3.6, “Building RPMs from
   Source” <#sect-source-buildrpm>`__ or `Section 3.5, “Building DEB
   packages” <#sect-source-builddebs>`__ as appropriate.

#. 

   This step is required only for installations where XenServer is
   installed on the hypervisor hosts.

   Download vhd-util from
   `vhd-util <http://download.cloud.com.s3.amazonaws.com/tools/vhd-util>`__

   Copy vhd-util to
   /usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver.

#. 

   Ensure that necessary services are started and set to start on boot.

   .. code:: bash

       # service rpcbind start
       # service nfs start
       # chkconfig nfs on
       # chkconfig rpcbind on

#. 

   Configure the database client. Note the absence of the --deploy-as
   argument in this case. (For more details about the arguments to this
   command, see `Section 4.5.4.2, “Install the Database on a Separate
   Node” <#management-server-install-db-external>`__.)

   .. code:: bash

       # cloudstack-setup-databases cloud:dbpassword@dbhost -e encryption_type -m management_server_key -k database_key -i management_server_ip

#. 

   Configure the OS and start the Management Server:

   .. code:: bash

       # cloudstack-setup-management

   The Management Server on this node should now be running.

#. 

   Repeat these steps on each additional Management Server.

#. 

   Be sure to configure a load balancer for the Management Servers. See
   `Section 13.6, “Management Server Load
   Balancing” <#management-server-lb>`__.

Prepare the System VM Template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Secondary storage must be seeded with a template that is used for
CloudStack system VMs.

.. note:: When copying and pasting a command, be sure the command has pasted as a
          single line before executing. Some document viewers may introduce unwanted line breaks in copied text.

#. 

   On the Management Server, run one or more of the following
   cloud-install-sys-tmplt commands to retrieve and decompress the
   system VM template. Run the command for each hypervisor type that you
   expect end users to run in this Zone.

   If your secondary storage mount point is not named /mnt/secondary,
   substitute your own mount point name.

   If you set the CloudStack database encryption type to "web" when you
   set up the database, you must now add the parameter -s
   <management-server-secret-key>. See `Section 4.5.5, “About Password
   and Key Encryption” <#about-password-encryption>`__.

   This process will require approximately 5 GB of free space on the
   local file system and up to 30 minutes each time it runs.

   -  

      For Hyper-V

      .. code:: bash

          # /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt/secondary -u http://download.cloud.com/templates/4.3/systemvm64template-2013-12-23-hyperv.vhd.bz2 -h hyperv -s <optional-management-server-secret-key> -F

   -  

      For XenServer:

      .. code:: bash

          # /usr/lib64/cloud/common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt/secondary -u http://download.cloud.com/templates/acton/acton-systemvm-02062012.vhd.bz2 -h xenserver -s <optional-management-server-secret-key> -F

   -  

      For vSphere:

      .. code:: bash

          # /usr/lib64/cloud/common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt/secondary -u http://download.cloud.com/templates/burbank/burbank-systemvm-08012012.ova -h vmware -s <optional-management-server-secret-key>  -F

   -  

      For KVM:

      .. code:: bash

          # /usr/lib64/cloud/common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt/secondary -u http://download.cloud.com/templates/acton/acton-systemvm-02062012.qcow2.bz2 -h kvm -s <optional-management-server-secret-key> -F

   -  

      For LXC:

      .. code:: bash

          # /usr/lib64/cloud/common/scripts/storage/secondary/cloud-install-sys-tmplt -m /mnt/secondary -u http://download.cloud.com/templates/acton/acton-systemvm-02062012.qcow2.bz2 -h lxc -s <optional-management-server-secret-key> -F

   On Ubuntu, use the following path instead:

   .. code:: bash

       # /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt

#. 

   If you are using a separate NFS server, perform this step. If you are
   using the Management Server as the NFS server, you MUST NOT perform
   this step.

   When the script has finished, unmount secondary storage and remove
   the created directory.

   .. code:: bash

       # umount /mnt/secondary
       # rmdir /mnt/secondary

#. 

   Repeat these steps for each secondary storage server.

Installation Complete! Next Steps
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Congratulations! You have now installed CloudStack Management Server and
the database it uses to persist system data.

|installation-complete.png: Finished installs with single Management
Server and multiple Management Servers|

What should you do next?

-  

   Even without adding any cloud infrastructure, you can run the UI to
   get a feel for what's offered and how you will interact with
   CloudStack on an ongoing basis. See Log In to the UI.

-  

   When you're ready, add the cloud infrastructure and try running some
   virtual machines on it, so you can watch how CloudStack manages the
   infrastructure. See Provision Your Cloud Infrastructure.


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
