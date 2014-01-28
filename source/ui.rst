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

User Interface
==============

Log In to the UI
----------------

CloudStack provides a web-based UI that can be used by both
administrators and end users. The appropriate version of the UI is
displayed depending on the credentials used to log in. The UI is
available in popular browsers including IE7, IE8, IE9, Firefox 3.5+,
Firefox 4, Safari 4, and Safari 5. The URL is: (substitute your own
management server IP address)

::

    http://<management-server-ip-address>:8080/client

On a fresh Management Server installation, a guided tour splash screen
appears. On later visits, you’ll see a login screen where you specify
the following to proceed to your Dashboard:

Username -> The user ID of your account. The default username is admin.

Password -> The password associated with the user ID. The password for the default
username is password.

Domain -> If you are a root user, leave this field blank.

If you are a user in the sub-domains, enter the full path to the domain,
excluding the root domain.

For example, suppose multiple levels are created under the root domain,
such as Comp1/hr. The users in the Comp1 domain should enter Comp1 in
the Domain field, whereas the users in the Comp1/sales domain should
enter Comp1/sales.

For more guidance about the choices that appear when you log in to this
UI, see Logging In as the Root Administrator.

End User's UI Overview
~~~~~~~~~~~~~~~~~~~~~~

The CloudStack UI helps users of cloud infrastructure to view and use
their cloud resources, including virtual machines, templates and ISOs,
data volumes and snapshots, guest networks, and IP addresses. If the
user is a member or administrator of one or more CloudStack projects,
the UI can provide a project-oriented view.

Root Administrator's UI Overview
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The CloudStack UI helps the CloudStack administrator provision, view,
and manage the cloud infrastructure, domains, user accounts, projects,
and configuration settings. The first time you start the UI after a
fresh Management Server installation, you can choose to follow a guided
tour to provision your cloud infrastructure. On subsequent logins, the
dashboard of the logged-in user appears. The various links in this
screen and the navigation bar on the left provide access to a variety of
administrative functions. The root administrator can also use the UI to
perform all the same tasks that are present in the end-user’s UI.

Logging In as the Root Administrator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After the Management Server software is installed and running, you can
run the CloudStack user interface. This UI is there to help you
provision, view, and manage your cloud infrastructure.

#. 

   Open your favorite Web browser and go to this URL. Substitute the IP
   address of your own Management Server:

   ::

       http://<management-server-ip-address>:8080/client

   After logging into a fresh Management Server installation, a guided
   tour splash screen appears. On later visits, you’ll be taken directly
   into the Dashboard.

#. 

   If you see the first-time splash screen, choose one of the following.

   -  

      **Continue with basic setup.** Choose this if you're just trying
      CloudStack, and you want a guided walkthrough of the simplest
      possible configuration so that you can get started right away.
      We'll help you set up a cloud with the following features: a
      single machine that runs CloudStack software and uses NFS to
      provide storage; a single machine running VMs under the XenServer
      or KVM hypervisor; and a shared public network.

      The prompts in this guided tour should give you all the
      information you need, but if you want just a bit more detail, you
      can follow along in the Trial Installation Guide.

   -  

      **I have used CloudStack before.** Choose this if you have already
      gone through a design phase and planned a more sophisticated
      deployment, or you are ready to start scaling up a trial cloud
      that you set up earlier with the basic setup screens. In the
      Administrator UI, you can start using the more powerful features
      of CloudStack, such as advanced VLAN networking, high
      availability, additional network elements such as load balancers
      and firewalls, and support for multiple hypervisors including
      Citrix XenServer, KVM, and VMware vSphere.

      The root administrator Dashboard appears.

#. 

   You should set a new root administrator password. If you chose basic
   setup, you’ll be prompted to create a new password right away. If you
   chose experienced user, use the steps in `Section 5.1.4, “Changing
   the Root Password” <#changing-root-password>`__.

.. warning:: You are logging in as the root administrator. This account manages the CloudStack deployment, including physical infrastructure. The root administrator can modify configuration settings to change basic functionality, create or delete user accounts, and take many actions that should be performed only by an authorized person. Please change the default password to a new, unique password.

Changing the Root Password
~~~~~~~~~~~~~~~~~~~~~~~~~~

During installation and ongoing cloud administration, you will need to
log in to the UI as the root administrator. The root administrator
account manages the CloudStack deployment, including physical
infrastructure. The root administrator can modify configuration settings
to change basic functionality, create or delete user accounts, and take
many actions that should be performed only by an authorized person. When
first installing CloudStack, be sure to change the default password to a
new, unique value.

#. 

   Open your favorite Web browser and go to this URL. Substitute the IP
   address of your own Management Server:

   ::

       http://<management-server-ip-address>:8080/client

#. 

   Log in to the UI using the current root user ID and password. The
   default is admin, password.

#. 

   Click Accounts.

#. 

   Click the admin account name.

#. 

   Click View Users.

#. 

   Click the admin user name.

#. 

   Click the Change Password button. |change-password.png: button to
   change a user's password|

#. 

   Type the new password, and click OK.

Using SSH Keys for Authentication
---------------------------------

In addition to the username and password authentication, CloudStack
supports using SSH keys to log in to the cloud infrastructure for
additional security. You can use the createSSHKeyPair API to generate
the SSH keys.

Because each cloud user has their own SSH key, one cloud user cannot log
in to another cloud user's instances unless they share their SSH key
files. Using a single SSH key pair, you can manage multiple instances.

Creating an Instance Template that Supports SSH Keys
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create an instance template that supports SSH Keys.

#. 

   Create a new instance by using the template provided by cloudstack.

   For more information on creating a new instance, see

#. 

   Download the cloudstack script from `The SSH Key Gen
   Script <http://sourceforge.net/projects/cloudstack/files/SSH%20Key%20Gen%20Script/>`__\ to
   the instance you have created.

   ::

       wget http://downloads.sourceforge.net/project/cloudstack/SSH%20Key%20Gen%20Script/cloud-set-guest-sshkey.in?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fcloudstack%2Ffiles%2FSSH%2520Key%2520Gen%2520Script%2F&ts=1331225219&use_mirror=iweb

#. 

   Copy the file to /etc/init.d.

   ::

       cp cloud-set-guest-sshkey.in /etc/init.d/

#. 

   Give the necessary permissions on the script:

   ::

       chmod +x /etc/init.d/cloud-set-guest-sshkey.in

#. 

   Run the script while starting up the operating system:

   ::

       chkconfig --add cloud-set-guest-sshkey.in

#. 

   Stop the instance.

Creating the SSH Keypair
~~~~~~~~~~~~~~~~~~~~~~~~

You must make a call to the createSSHKeyPair api method. You can either
use the CloudStack Python API library or the curl commands to make the
call to the cloudstack api.

For example, make a call from the cloudstack server to create a SSH
keypair called "keypair-doc" for the admin account in the root domain:

Note


Ensure that you adjust these values to meet your needs. If you are
making the API call from a different server, your URL/PORT will be
different, and you will need to use the API keys.

#. 

   Run the following curl command:

   ::

       curl --globoff "http://localhost:8096/?command=createSSHKeyPair&name=keypair-doc&account=admin&domainid=5163440e-c44b-42b5-9109-ad75cae8e8a2"

   The output is something similar to what is given below:

   ::

       <?xml version="1.0" encoding="ISO-8859-1"?><createsshkeypairresponse cloud-stack-version="3.0.0.20120228045507"><keypair><name>keypair-doc</name><fingerprint>f6:77:39:d5:5e:77:02:22:6a:d8:7f:ce:ab:cd:b3:56</fingerprint><privatekey>-----BEGIN RSA PRIVATE KEY-----
       MIICXQIBAAKBgQCSydmnQ67jP6lNoXdX3noZjQdrMAWNQZ7y5SrEu4wDxplvhYci
       dXYBeZVwakDVsU2MLGl/K+wefwefwefwefwefJyKJaogMKn7BperPD6n1wIDAQAB
       AoGAdXaJ7uyZKeRDoy6wA0UmF0kSPbMZCR+UTIHNkS/E0/4U+6lhMokmFSHtu
       mfDZ1kGGDYhMsdytjDBztljawfawfeawefawfawfawQQDCjEsoRdgkduTy
       QpbSGDIa11Jsc+XNDx2fgRinDsxXI/zJYXTKRhSl/LIPHBw/brW8vzxhOlSOrwm7
       VvemkkgpAkEAwSeEw394LYZiEVv395ar9MLRVTVLwpo54jC4tsOxQCBlloocK
       lYaocpk0yBqqOUSBawfIiDCuLXSdvBo1Xz5ICTM19vgvEp/+kMuECQBzm
       nVo8b2Gvyagqt/KEQo8wzH2THghZ1qQ1QRhIeJG2aissEacF6bGB2oZ7Igim5L14
       4KR7OeEToyCLC2k+02UCQQCrniSnWKtDVoVqeK/zbB32JhW3Wullv5p5zUEcd
       KfEEuzcCUIxtJYTahJ1pvlFkQ8anpuxjSEDp8x/18bq3
       -----END RSA PRIVATE KEY-----
       </privatekey></keypair></createsshkeypairresponse>

#. 

   Copy the key data into a file. The file looks like this:

   ::

       -----BEGIN RSA PRIVATE KEY-----
       MIICXQIBAAKBgQCSydmnQ67jP6lNoXdX3noZjQdrMAWNQZ7y5SrEu4wDxplvhYci
       dXYBeZVwakDVsU2MLGl/K+wefwefwefwefwefJyKJaogMKn7BperPD6n1wIDAQAB
       AoGAdXaJ7uyZKeRDoy6wA0UmF0kSPbMZCR+UTIHNkS/E0/4U+6lhMokmFSHtu
       mfDZ1kGGDYhMsdytjDBztljawfawfeawefawfawfawQQDCjEsoRdgkduTy
       QpbSGDIa11Jsc+XNDx2fgRinDsxXI/zJYXTKRhSl/LIPHBw/brW8vzxhOlSOrwm7
       VvemkkgpAkEAwSeEw394LYZiEVv395ar9MLRVTVLwpo54jC4tsOxQCBlloocK
       lYaocpk0yBqqOUSBawfIiDCuLXSdvBo1Xz5ICTM19vgvEp/+kMuECQBzm
       nVo8b2Gvyagqt/KEQo8wzH2THghZ1qQ1QRhIeJG2aissEacF6bGB2oZ7Igim5L14
       4KR7OeEToyCLC2k+02UCQQCrniSnWKtDVoVqeK/zbB32JhW3Wullv5p5zUEcd
       KfEEuzcCUIxtJYTahJ1pvlFkQ8anpuxjSEDp8x/18bq3
       -----END RSA PRIVATE KEY-----

#. 

   Save the file.

Creating an Instance
~~~~~~~~~~~~~~~~~~~~

After you save the SSH keypair file, you must create an instance by
using the template that you created at `Section 5.2.1, “ Creating an
Instance Template that Supports SSH Keys” <#create-ssh-template>`__.
Ensure that you use the same SSH key name that you created at
`Section 5.2.2, “Creating the SSH Keypair” <#create-ssh-keypair>`__.

.. note:: You cannot create the instance by using the GUI at this time and associate the instance with the newly created SSH keypair.

A sample curl command to create a new instance is:

::

    curl --globoff http://localhost:<port number>/?command=deployVirtualMachine\&zoneId=1\&serviceOfferingId=18727021-7556-4110-9322-d625b52e0813\&templateId=e899c18a-ce13-4bbf-98a9-625c5026e0b5\&securitygroupids=ff03f02f-9e3b-48f8-834d-91b822da40c5\&account=admin\&domainid=1\&keypair=keypair-doc

Substitute the template, service offering and security group IDs (if you
are using the security group feature) that are in your cloud
environment.

Logging In Using the SSH Keypair
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To test your SSH key generation is successful, check whether you can log
in to the cloud setup.

For example, from a Linux OS, run:

::

    ssh -i ~/.ssh/keypair-doc <ip address>

The -i parameter tells the ssh client to use a ssh key found at
~/.ssh/keypair-doc.

Resetting SSH Keys
~~~~~~~~~~~~~~~~~~

With the API command resetSSHKeyForVirtualMachine, a user can set or
reset the SSH keypair assigned to a virtual machine. A lost or
compromised SSH keypair can be changed, and the user can access the VM
by using the new keypair. Just create or register a new keypair, then
call resetSSHKeyForVirtualMachine.


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
