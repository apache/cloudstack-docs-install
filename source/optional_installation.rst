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

Additional Installation Options
===============================

The next few sections describe CloudStack features above and beyond the
basic deployment options.

Installing the Usage Server (Optional)
--------------------------------------

You can optionally install the Usage Server once the Management Server
is configured properly. The Usage Server takes data from the events in
the system and enables usage-based billing for accounts.

When multiple Management Servers are present, the Usage Server may be
installed on any number of them. The Usage Servers will coordinate usage
processing. A site that is concerned about availability should install
Usage Servers on at least two Management Servers.

Requirements for Installing the Usage Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  

   The Management Server must be running when the Usage Server is
   installed.

-  

   The Usage Server must be installed on the same server as a Management
   Server.

Steps to Install the Usage Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. 

   Run ./install.sh.

   .. code:: bash

       # ./install.sh

   You should see a few messages as the installer prepares, followed by
   a list of choices.

#. 

   Choose "S" to install the Usage Server.

   .. code:: bash

          > S

#. 

   Once installed, start the Usage Server with the following command.

   .. code:: bash

       # service cloudstack-usage start

The Administration Guide discusses further configuration of the Usage
Server.

SSL (Optional)
--------------

CloudStack provides HTTP access in its default installation. There are a
number of technologies and sites which choose to implement SSL. As a
result, we have left CloudStack to expose HTTP under the assumption that
a site will implement its typical practice.

CloudStack uses Tomcat as its servlet container. For sites that would
like CloudStack to terminate the SSL session, Tomcat’s SSL access may be
enabled. Tomcat SSL configuration is described at
http://tomcat.apache.org/tomcat-6.0-doc/ssl-howto.html.

Database Replication (Optional)
-------------------------------

CloudStack supports database replication from one MySQL node to another.
This is achieved using standard MySQL replication. You may want to do
this as insurance against MySQL server or storage loss. MySQL
replication is implemented using a master/slave model. The master is the
node that the Management Servers are configured to use. The slave is a
standby node that receives all write operations from the master and
applies them to a local, redundant copy of the database. The following
steps are a guide to implementing MySQL replication.

.. note:: Creating a replica is not a backup solution. You should develop a backup
procedure for the MySQL data that is distinct from replication.

#. 

   Ensure that this is a fresh install with no data in the master.

#. 

   Edit my.cnf on the master and add the following in the [mysqld]
   section below datadir.

   .. code:: bash

       log_bin=mysql-bin
       server_id=1

   The server\_id must be unique with respect to other servers. The
   recommended way to achieve this is to give the master an ID of 1 and
   each slave a sequential number greater than 1, so that the servers
   are numbered 1, 2, 3, etc.

#. 

   Restart the MySQL service. On RHEL/CentOS systems, use:

   .. code:: bash

       # service mysqld restart

   On Debian/Ubuntu systems, use:

   .. code:: bash

       # service mysql restart

#. 

   Create a replication account on the master and give it privileges. We
   will use the "cloud-repl" user with the password "password". This
   assumes that master and slave run on the 172.16.1.0/24 network.

   .. code:: bash

       # mysql -u root
       mysql> create user 'cloud-repl'@'172.16.1.%' identified by 'password';
       mysql> grant replication slave on *.* TO 'cloud-repl'@'172.16.1.%';
       mysql> flush privileges;
       mysql> flush tables with read lock;

#. 

   Leave the current MySQL session running.

#. 

   In a new shell start a second MySQL session.

#. 

   Retrieve the current position of the database.

   .. code:: bash

       # mysql -u root
       mysql> show master status;
       +------------------+----------+--------------+------------------+
       | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
       +------------------+----------+--------------+------------------+
       | mysql-bin.000001 |      412 |              |                  |
       +------------------+----------+--------------+------------------+

#. 

   Note the file and the position that are returned by your instance.

#. 

   Exit from this session.

#. 

   Complete the master setup. Returning to your first session on the
   master, release the locks and exit MySQL.

   .. code:: bash

       mysql> unlock tables;

#. 

   Install and configure the slave. On the slave server, run the
   following commands.

   .. code:: bash

       # yum install mysql-server
       # chkconfig mysqld on

#. 

   Edit my.cnf and add the following lines in the [mysqld] section below
   datadir.

   .. code:: bash

       server_id=2
       innodb_rollback_on_timeout=1
       innodb_lock_wait_timeout=600

#. 

   Restart MySQL. Use "mysqld" on RHEL/CentOS systems:

   .. code:: bash

       # service mysqld restart

   On Ubuntu/Debian systems use "mysql."

   .. code:: bash

       # service mysql restart

#. 

   Instruct the slave to connect to and replicate from the master.
   Replace the IP address, password, log file, and position with the
   values you have used in the previous steps.

   .. code:: bash

       mysql> change master to
           -> master_host='172.16.1.217',
           -> master_user='cloud-repl',
           -> master_password='password',
           -> master_log_file='mysql-bin.000001',
           -> master_log_pos=412;

#. 

   Then start replication on the slave.

   .. code:: bash

       mysql> start slave;

#. 

   Optionally, open port 3306 on the slave as was done on the master
   earlier.

   This is not required for replication to work. But if you choose not
   to do this, you will need to do it when failover to the replica
   occurs.

Failover
~~~~~~~~

This will provide for a replicated database that can be used to
implement manual failover for the Management Servers. CloudStack
failover from one MySQL instance to another is performed by the
administrator. In the event of a database failure you should:

#. 

   Stop the Management Servers (via service cloudstack-management stop).

#. 

   Change the replica's configuration to be a master and restart it.

#. 

   Ensure that the replica's port 3306 is open to the Management
   Servers.

#. 

   Make a change so that the Management Server uses the new database.
   The simplest process here is to put the IP address of the new
   database server into each Management Server's
   /etc/cloudstack/management/db.properties.

#. 

   Restart the Management Servers:

   .. code:: bash

       # service cloudstack-management start


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
