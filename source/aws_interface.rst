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


Amazon Web Services Interface
============================

Amazon Web Services Compatible Interface
----------------------------------------

CloudStack can translate Amazon Web Services (AWS) API calls to native
CloudStack API calls so that users can continue using existing
AWS-compatible tools. This translation service runs as a separate web
application in the same tomcat server as the management server of
CloudStack, listening on a different port. The Amazon Web Services (AWS)
compatible interface provides the EC2 SOAP and Query APIs as well as the
S3 REST API.

.. note:: This service was previously enabled by separate software called
CloudBridge. It is now fully integrated with the CloudStack management
server.

.. warning:: The compatible interface for the EC2 Query API and the S3 API are Work
In Progress. The S3 compatible API offers a way to store data on the
management server file system, it is not an implementation of the S3
backend.

Limitations

-  

   Supported only in zones that use basic networking.

-  

   Available in fresh installations of CloudStack. Not available through
   upgrade of previous versions.

-  

   Features such as Elastic IP (EIP) and Elastic Load Balancing (ELB)
   are only available in an infrastructure with a Citrix NetScaler
   device. Users accessing a Zone with a NetScaler device will need to
   use a NetScaler-enabled network offering (DefaultSharedNetscalerEIP
   and ELBNetworkOffering).

Supported API Version
---------------------

-  

   The EC2 interface complies with Amazon's WDSL version dated November
   15, 2010, available at
   `http://ec2.amazonaws.com/doc/2010-11-15/ <http://ec2.amazonaws.com/doc/2010-11-15/>`__.

-  

   The interface is compatible with the EC2 command-line tools *EC2
   tools v. 1.3.6230*, which can be downloaded at
   `http://s3.amazonaws.com/ec2-downloads/ec2-api-tools-1.3-62308.zip <http://s3.amazonaws.com/ec2-downloads/ec2-api-tools-1.3-62308.zip>`__.

.. note:: Work is underway to support a more recent version of the EC2 API

Enabling the EC2 and S3 Compatible Interface
--------------------------------------------

The software that provides AWS API compatibility is installed along with
CloudStack. You must enable the services and perform some setup steps
prior to using it.

#. 

   Set the global configuration parameters for each service to true. See
   `Chapter 7, *Setting Configuration Parameters* <#global-config>`__.

#. 

   Create a set of CloudStack service offerings with names that match
   the Amazon service offerings. You can do this through the CloudStack
   UI as described in the Administration Guide.

   .. warning:: Be sure you have included the Amazon default service offering,
   m1.small. As well as any EC2 instance types that you will use.

#. 

   If you did not already do so when you set the configuration parameter
   in step `1 <#set-global-config>`__, restart the Management Server.

::

       # service cloudstack-management restart

The following sections provides details to perform these steps

Enabling the Services
~~~~~~~~~~~~~~~~~~~~~

To enable the EC2 and S3 compatible services you need to set the
configuration variables *enable.ec2.api* and *enable.s3.api* to true.
You do not have to enable both at the same time. Enable the ones you
need. This can be done via the CloudStack GUI by going in *Global
Settings* or via the API.

The snapshot below shows you how to use the GUI to enable these services

|Use the GUI to set the configuration variable to true|

Using the CloudStack API, the easiest is to use the so-called
integration port on which you can make unauthenticated calls. In Global
Settings set the port to 8096 and subsequently call the
*updateConfiguration* method. The following urls shows you how:

::

    http://localhost:8096/client/api?command=updateConfiguration&name=enable.ec2.api&value=true
    http://localhost:8096/client/api?command=updateConfiguration&name=enable.ec2.api&value=true

Once you have enabled the services, restart the server.

Creating EC2 Compatible Service Offerings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will also need to define compute service offerings with names
compatible with the `Amazon EC2 instance
types <http://aws.amazon.com/ec2/instance-types/>`__ API names (e.g
m1.small,m1.large). This can be done via the CloudStack GUI. Go under
*Service Offerings* select *Compute offering* and either create a new
compute offering or modify an existing one, ensuring that the name
matches an EC2 instance type API name. The snapshot below shows you how:

|Use the GUI to set the name of a compute service offering to an EC2
instance type API name.|

Modifying the AWS API Port
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: (Optional) The AWS API listens for requests on port 7080. If you prefer
AWS API to listen on another port, you can change it as follows:

#. 

   Edit the files /etc/cloudstack/management/server.xml,
   /etc/cloudstack/management/server-nonssl.xml, and
   /etc/cloudstack/management/server-ssl.xml.

#. 

   In each file, find the tag <Service name="Catalina7080">. Under this
   tag, locate <Connector executor="tomcatThreadPool-internal" port=
   ....<.

#. 

   Change the port to whatever port you want to use, then save the
   files.

#. 

   Restart the Management Server.

If you re-install CloudStack, you will have to re-enable the services
and if need be update the port.

AWS API User Setup
------------------

In general, users need not be aware that they are using a translation
service provided by CloudStack. They only need to send AWS API calls to
CloudStack's endpoint, and it will translate the calls to the native
CloudStack API. Users of the Amazon EC2 compatible interface will be
able to keep their existing EC2 tools and scripts and use them with
their CloudStack deployment, by specifying the endpoint of the
management server and using the proper user credentials. In order to do
this, each user must perform the following configuration steps:

-  

   Generate user credentials.

-  

   Register with the service.

-  

   For convenience, set up environment variables for the EC2 SOAP
   command-line tools.


AWS API Command-Line Tools Setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use the EC2 command-line tools, the user must perform these steps:

#. 

   Be sure you have the right version of EC2 Tools. The supported
   version is available at
   `http://s3.amazonaws.com/ec2-downloads/ec2-api-tools-1.3-62308.zip <http://s3.amazonaws.com/ec2-downloads/ec2-api-tools-1.3-62308.zip>`__.

#. 

   Set up the EC2 environment variables. This can be done every time you
   use the service or you can set them up in the proper shell profile.
   Replace the endpoint (i.e EC2\_URL) with the proper address of your
   CloudStack management server and port. In a bash shell do the
   following.

   .. code:: bash

                         $ export EC2_CERT=/path/to/cert.pem
                         $ export EC2_PRIVATE_KEY=/path/to/private_key.pem
                         $ export EC2_URL=http://localhost:7080/awsapi
                         $ export EC2_HOME=/path/to/EC2_tools_directory

Using Timeouts to Ensure AWS API Command Completion
---------------------------------------------------

The Amazon EC2 command-line tools have a default connection timeout.
When used with CloudStack, a longer timeout might be needed for some
commands. If you find that commands are not completing due to timeouts,
you can specify a custom timeouts. You can add the following optional
command-line parameters to any CloudStack-supported EC2 command:

.. code:: bash

    --connection-timeout TIMEOUT

Specifies a connection timeout (in seconds). Example:

.. code:: bash

    --connection-timeout 30

.. code:: bash

    --request-timeout TIMEOUT

Specifies a request timeout (in seconds). Example:

.. code:: bash

    --request-timeout 45

Example:

.. code:: bash

    ec2-run-instances 2 –z us-test1 –n 1-3 --connection-timeout 120 --request-timeout 120

.. note:: The timeouts optional arguments are not specific to CloudStack.

Supported AWS API Calls
-----------------------

The following Amazon EC2 commands are supported by CloudStack when the
AWS API compatible interface is enabled. For a few commands, there are
differences between the CloudStack and Amazon EC2 versions, and these
differences are noted. The underlying SOAP call for each command is also
given, for those who have built tools using those calls.

Table 1. Elastic IP API mapping
                                  

+---------------------------+-----------------------+-------------------------+
| EC2 command               | SOAP call             | CloudStack API call     |
+===========================+=======================+=========================+
| ec2-allocate-address      | AllocateAddress       | associateIpAddress      |
+---------------------------+-----------------------+-------------------------+
| ec2-associate-address     | AssociateAddress      | enableStaticNat         |
+---------------------------+-----------------------+-------------------------+
| ec2-describe-addresses    | DescribeAddresses     | listPublicIpAddresses   |
+---------------------------+-----------------------+-------------------------+
| ec2-diassociate-address   | DisassociateAddress   | disableStaticNat        |
+---------------------------+-----------------------+-------------------------+
| ec2-release-address       | ReleaseAddress        | disassociateIpAddress   |
+---------------------------+-----------------------+-------------------------+

| 

Table 2. Availability Zone API mapping
                                         

+-----------------------------------+-----------------------------+-----------------------+
| EC2 command                       | SOAP call                   | CloudStack API call   |
+===================================+=============================+=======================+
| ec2-describe-availability-zones   | DescribeAvailabilityZones   | listZones             |
+-----------------------------------+-----------------------------+-----------------------+

| 

Table 3. Images API mapping
                              

+-----------------------+-------------------+-----------------------+
| EC2 command           | SOAP call         | CloudStack API call   |
+=======================+===================+=======================+
| ec2-create-image      | CreateImage       | createTemplate        |
+-----------------------+-------------------+-----------------------+
| ec2-deregister        | DeregisterImage   | DeleteTemplate        |
+-----------------------+-------------------+-----------------------+
| ec2-describe-images   | DescribeImages    | listTemplates         |
+-----------------------+-------------------+-----------------------+
| ec2-register          | RegisterImage     | registerTemplate      |
+-----------------------+-------------------+-----------------------+

| 

Table 4. Image Attributes API mapping
                                        

+--------------------------------+--------------------------+-----------------------------+
| EC2 command                    | SOAP call                | CloudStack API call         |
+================================+==========================+=============================+
| ec2-describe-image-attribute   | DescribeImageAttribute   | listTemplatePermissions     |
+--------------------------------+--------------------------+-----------------------------+
| ec2-modify-image-attribute     | ModifyImageAttribute     | updateTemplatePermissions   |
+--------------------------------+--------------------------+-----------------------------+
| ec2-reset-image-attribute      | ResetImageAttribute      | updateTemplatePermissions   |
+--------------------------------+--------------------------+-----------------------------+

| 

Table 5. Instances API mapping
                                 

+---------------------------+----------------------+-------------------------+
| EC2 command               | SOAP call            | CloudStack API call     |
+===========================+======================+=========================+
| ec2-describe-instances    | DescribeInstances    | listVirtualMachines     |
+---------------------------+----------------------+-------------------------+
| ec2-run-instances         | RunInstances         | deployVirtualMachine    |
+---------------------------+----------------------+-------------------------+
| ec2-reboot-instances      | RebootInstances      | rebootVirtualMachine    |
+---------------------------+----------------------+-------------------------+
| ec2-start-instances       | StartInstances       | startVirtualMachine     |
+---------------------------+----------------------+-------------------------+
| ec2-stop-instances        | StopInstances        | stopVirtualMachine      |
+---------------------------+----------------------+-------------------------+
| ec2-terminate-instances   | TerminateInstances   | destroyVirtualMachine   |
+---------------------------+----------------------+-------------------------+

| 

Table 6. Instance Attributes Mapping
                                       

+-----------------------------------+-----------------------------+-----------------------+
| EC2 command                       | SOAP call                   | CloudStack API call   |
+===================================+=============================+=======================+
| ec2-describe-instance-attribute   | DescribeInstanceAttribute   | listVirtualMachines   |
+-----------------------------------+-----------------------------+-----------------------+

| 

Table 7. Keys Pairs Mapping
                              

+-------------------------+--------------------+-----------------------+
| EC2 command             | SOAP call          | CloudStack API call   |
+=========================+====================+=======================+
| ec2-add-keypair         | CreateKeyPair      | createSSHKeyPair      |
+-------------------------+--------------------+-----------------------+
| ec2-delete-keypair      | DeleteKeyPair      | deleteSSHKeyPair      |
+-------------------------+--------------------+-----------------------+
| ec2-describe-keypairs   | DescribeKeyPairs   | listSSHKeyPairs       |
+-------------------------+--------------------+-----------------------+
| ec2-import-keypair      | ImportKeyPair      | registerSSHKeyPair    |
+-------------------------+--------------------+-----------------------+

| 

Table 8. Passwords API Mapping
                                 

+--------------------+-------------------+-----------------------+
| EC2 command        | SOAP call         | CloudStack API call   |
+====================+===================+=======================+
| ec2-get-password   | GetPasswordData   | getVMPassword         |
+--------------------+-------------------+-----------------------+

| 

Table 9. Security Groups API Mapping
                                       

+----------------------+---------------------------------+---------------------------------+
| EC2 command          | SOAP call                       | CloudStack API call             |
+======================+=================================+=================================+
| ec2-authorize        | AuthorizeSecurityGroupIngress   | authorizeSecurityGroupIngress   |
+----------------------+---------------------------------+---------------------------------+
| ec2-add-group        | CreateSecurityGroup             | createSecurityGroup             |
+----------------------+---------------------------------+---------------------------------+
| ec2-delete-group     | DeleteSecurityGroup             | deleteSecurityGroup             |
+----------------------+---------------------------------+---------------------------------+
| ec2-describe-group   | DescribeSecurityGroups          | listSecurityGroups              |
+----------------------+---------------------------------+---------------------------------+
| ec2-revoke           | RevokeSecurityGroupIngress      | revokeSecurityGroupIngress      |
+----------------------+---------------------------------+---------------------------------+

| 

Table 10. Snapshots API Mapping
                                  

+--------------------------+---------------------+-----------------------+
| EC2 command              | SOAP call           | CloudStack API call   |
+==========================+=====================+=======================+
| ec2-create-snapshot      | CreateSnapshot      | createSnapshot        |
+--------------------------+---------------------+-----------------------+
| ec2-delete-snapshot      | DeleteSnapshot      | deleteSnapshot        |
+--------------------------+---------------------+-----------------------+
| ec2-describe-snapshots   | DescribeSnapshots   | listSnapshots         |
+--------------------------+---------------------+-----------------------+

| 

Table 11. Volumes API Mapping
                                

+-----------------------+------------------+-----------------------+
| EC2 command           | SOAP call        | CloudStack API call   |
+=======================+==================+=======================+
| ec2-attach-volume     | AttachVolume     | attachVolume          |
+-----------------------+------------------+-----------------------+
| ec2-create-volume     | CreateVolume     | createVolume          |
+-----------------------+------------------+-----------------------+
| ec2-delete-volume     | DeleteVolume     | deleteVolume          |
+-----------------------+------------------+-----------------------+
| ec2-describe-volume   | DescribeVolume   | listVolumes           |
+-----------------------+------------------+-----------------------+
| ec2-detach-volume     | DetachVolume     | detachVolume          |
+-----------------------+------------------+-----------------------+

| 

Examples
--------

There are many tools available to interface with a AWS compatible API.
In this section we provide a few examples that users of CloudStack can
build upon.

Boto Examples
~~~~~~~~~~~~~

Boto is one of them. It is a Python package available at
https://github.com/boto/boto. In this section we provide two examples of
Python scripts that use Boto and have been tested with the CloudStack
AWS API Interface.

First is an EC2 example. Replace the Access and Secret Keys with your
own and update the endpoint.

Example 1. An EC2 Boto example
                                 

.. code:: python

    #!/usr/bin/env python

    import sys
    import os
    import boto
    import boto.ec2

    region = boto.ec2.regioninfo.RegionInfo(name="ROOT",endpoint="localhost")
    apikey='GwNnpUPrO6KgIdZu01z_ZhhZnKjtSdRwuYd4DvpzvFpyxGMvrzno2q05MB0ViBoFYtdqKd'
    secretkey='t4eXLEYWw7chBhDlaKf38adCMSHx_wlds6JfSx3z9fSpSOm0AbP9Moj0oGIzy2LSC8iw'

    def main():
        '''Establish connection to EC2 cloud'''
            conn =boto.connect_ec2(aws_access_key_id=apikey,
                           aws_secret_access_key=secretkey,
                           is_secure=False,
                           region=region,
                           port=7080,
                           path="/awsapi",
                           api_version="2010-11-15")

            '''Get list of images that I own'''
        images = conn.get_all_images()
        print images
        myimage = images[0]
        '''Pick an instance type'''
        vm_type='m1.small'
        reservation = myimage.run(instance_type=vm_type,security_groups=['default'])

    if __name__ == '__main__':
        main()

| 

Second is an S3 example. Replace the Access and Secret keys with your
own, as well as the endpoint of the service. Be sure to also update the
file paths to something that exists on your machine.

Example 2. An S3 Boto Example
                                

.. code:: python

    #!/usr/bin/env python

    import sys
    import os
    from boto.s3.key import Key
    from boto.s3.connection import S3Connection
    from boto.s3.connection import OrdinaryCallingFormat

    apikey='ChOw-pwdcCFy6fpeyv6kUaR0NnhzmG3tE7HLN2z3OB_s-ogF5HjZtN4rnzKnq2UjtnHeg_yLA5gOw'
    secretkey='IMY8R7CJQiSGFk4cHwfXXN3DUFXz07cCiU80eM3MCmfLs7kusgyOfm0g9qzXRXhoAPCH-IRxXc3w'

    cf=OrdinaryCallingFormat()

    def main(): 
        '''Establish connection to S3 service'''
            conn =S3Connection(aws_access_key_id=apikey,aws_secret_access_key=secretkey, \
                              is_secure=False, \
                              host='localhost', \
                              port=7080, \
                              calling_format=cf, \
                              path="/awsapi/rest/AmazonS3")

            try:
                bucket=conn.create_bucket('cloudstack')
                k = Key(bucket)
                k.key = 'test'
                try:
                   k.set_contents_from_filename('/Users/runseb/Desktop/s3cs.py')
                except:
                   print 'could not write file'
                   pass
            except:
                bucket = conn.get_bucket('cloudstack')
                k = Key(bucket)
                k.key = 'test'
                try:
                   k.get_contents_to_filename('/Users/runseb/Desktop/foobar')
                except:
                   print 'Could not get file'
                   pass

            try:
               bucket1=conn.create_bucket('teststring')
               k=Key(bucket1)
               k.key('foobar')
               k.set_contents_from_string('This is my silly test')
            except:
               bucket1=conn.get_bucket('teststring')
               k = Key(bucket1)
               k.key='foobar'
               k.get_contents_as_string()
        
    if __name__ == '__main__':
        main()


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
