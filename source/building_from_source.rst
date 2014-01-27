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

Building from Source
====================

The official CloudStack release is always in source code form. You will
likely be able to find "convenience binaries," the source is the
canonical release. In this section, we'll cover acquiring the source
release and building that so that you can deploy it using Maven or
create Debian packages or RPMs.

Note that building and deploying directly from source is typically not
the most efficient way to deploy an IaaS. However, we will cover that
method as well as building RPMs or Debian packages for deploying
CloudStack.

The instructions here are likely version-specific. That is, the method
for building from source for the 4.0.x series is different from the
4.1.x series.

If you are working with a unreleased version of CloudStack, see the
INSTALL.md file in the top-level directory of the release.

Getting the release
-------------------

You can download the latest CloudStack release from the `Apache
CloudStack project download
page <http://incubator.apache.org/cloudstack/downloads.html>`__.

Prior releases are available via archive.apache.org as well. See the
downloads page for more information on archived releases.

You'll notice several links under the 'Latest release' section. A link
to a file ending in ``tar.bz2``, as well as a PGP/GPG signature, MD5,
and SHA512 file.

-  

   The ``tar.bz2`` file contains the Bzip2-compressed tarball with the
   source code.

-  

   The ``.asc`` file is a detached cryptographic signature that can be
   used to help verify the authenticity of the release.

-  

   The ``.md5`` file is an MD5 hash of the release to aid in verify the
   validity of the release download.

-  

   The ``.sha`` file is a SHA512 hash of the release to aid in verify
   the validity of the release download.

Verifying the downloaded release
--------------------------------

There are a number of mechanisms to check the authenticity and validity
of a downloaded release.

Getting the KEYS
~~~~~~~~~~~~~~~~

To enable you to verify the GPG signature, you will need to download the
`KEYS <http://www.apache.org/dist/incubator/cloudstack/KEYS>`__ file.

You next need to import those keys, which you can do by running:

.. code:: bash

    # gpg --import KEYS

GPG
~~~

The CloudStack project provides a detached GPG signature of the release.
To check the signature, run the following command:

.. code:: bash

    $ gpg --verify apache-cloudstack-4.0.0-incubating-src.tar.bz2.asc

If the signature is valid you will see a line of output that contains
'Good signature'.

MD5
~~~

In addition to the cryptographic signature, CloudStack has an MD5
checksum that you can use to verify the download matches the release.
You can verify this hash by executing the following command:

.. code:: bash

    $ gpg --print-md MD5 apache-cloudstack-4.0.0-incubating-src.tar.bz2 | diff - apache-cloudstack-4.0.0-incubating-src.tar.bz2.md5

If this successfully completes you should see no output. If there is any
output from them, then there is a difference between the hash you
generated locally and the hash that has been pulled from the server.

SHA512
~~~~~~

In addition to the MD5 hash, the CloudStack project provides a SHA512
cryptographic hash to aid in assurance of the validity of the downloaded
release. You can verify this hash by executing the following command:

.. code:: bash

    $ gpg --print-md SHA512 apache-cloudstack-4.0.0-incubating-src.tar.bz2 | diff - apache-cloudstack-4.0.0-incubating-src.tar.bz2.sha

If this command successfully completes you should see no output. If
there is any output from them, then there is a difference between the
hash you generated locally and the hash that has been pulled from the
server.

Prerequisites for building Apache CloudStack
--------------------------------------------

There are a number of prerequisites needed to build CloudStack. This
document assumes compilation on a Linux system that uses RPMs or DEBs
for package management.

You will need, at a minimum, the following to compile CloudStack:

#. 

   Maven (version 3)

#. 

   Java (OpenJDK 1.6 or Java 7/OpenJDK 1.7)

#. 

   Apache Web Services Common Utilities (ws-commons-util)

#. 

   MySQL

#. 

   MySQLdb (provides Python database API)

#. 

   Tomcat 6 (not 6.0.35)

#. 

   genisoimage

#. 

   rpmbuild or dpkg-dev

Extracting source
-----------------

Extracting the CloudStack release is relatively simple and can be done
with a single command as follows:

.. code:: bash

    $ tar -jxvf apache-cloudstack-4.1.0.src.tar.bz2

You can now move into the directory:

.. code:: bash

    $ cd ./apache-cloudstack-4.1.0-src

Building DEB packages
---------------------

In addition to the bootstrap dependencies, you'll also need to install
several other dependencies. Note that we recommend using Maven 3, which
is not currently available in 12.04.1 LTS. So, you'll also need to add a
PPA repository that includes Maven 3. After running the command
``add-apt-repository``, you will be prompted to continue and a GPG key
will be added.

.. code:: bash

    $ sudo apt-get update
    $ sudo apt-get install python-software-properties
    $ sudo add-apt-repository ppa:natecarlson/maven3
    $ sudo apt-get update
    $ sudo apt-get install ant debhelper openjdk-6-jdk tomcat6 libws-commons-util-java genisoimage python-mysqldb libcommons-codec-java libcommons-httpclient-java liblog4j1.2-java maven3

While we have defined, and you have presumably already installed the
bootstrap prerequisites, there are a number of build time prerequisites
that need to be resolved. CloudStack uses maven for dependency
resolution. You can resolve the buildtime depdencies for CloudStack by
running:

.. code:: bash

    $ mvn3 -P deps

Now that we have resolved the dependencies we can move on to building
CloudStack and packaging them into DEBs by issuing the following
command.

.. code:: bash

    $ dpkg-buildpackage -uc -us

This command will build the following debian packages. You should have
all of the following:

.. code:: bash

    cloudstack-common-4.2.0.amd64.deb
    cloudstack-management-4.2.0.amd64.deb
    cloudstack-agent-4.2.0.amd64.deb
    cloudstack-usage-4.2.0.amd64.deb
    cloudstack-awsapi-4.2.0.amd64.deb
    cloudstack-cli-4.2.0.amd64.deb
    cloudstack-docs-4.2.0.amd64.deb

Setting up an APT repo
~~~~~~~~~~~~~~~~~~~~~~

After you've created the packages, you'll want to copy them to a system
where you can serve the packages over HTTP. You'll create a directory
for the packages and then use ``dpkg-scanpackages`` to create
``Packages.gz``, which holds information about the archive structure.
Finally, you'll add the repository to your system(s) so you can install
the packages using APT.

The first step is to make sure that you have the **dpkg-dev** package
installed. This should have been installed when you pulled in the
**debhelper** application previously, but if you're generating
``Packages.gz`` on a different system, be sure that it's installed there
as well.

.. code:: bash

    $ sudo apt-get install dpkg-dev

The next step is to copy the DEBs to the directory where they can be
served over HTTP. We'll use ``/var/www/cloudstack/repo`` in the
examples, but change the directory to whatever works for you.

.. code:: bash

    sudo mkdir -p /var/www/cloudstack/repo/binary
    sudo cp *.deb /var/www/cloudstack/repo/binary
    sudo cd /var/www/cloudstack/repo/binary
    sudo dpkg-scanpackages . /dev/null | tee Packages | gzip -9 > Packages.gz

.. note:: You can safely ignore the warning about a missing override file.

Now you should have all of the DEB packages and ``Packages.gz`` in the
``binary`` directory and available over HTTP. (You may want to use
``wget`` or ``curl`` to test this before moving on to the next step.)

Configuring your machines to use the APT repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that we have created the repository, you need to configure your
machine to make use of the APT repository. You can do this by adding a
repository file under ``/etc/apt/sources.list.d``. Use your preferred
editor to create ``/etc/apt/sources.list.d/cloudstack.list`` with this
line:

.. code:: bash

    deb http://server.url/cloudstack/repo binary ./

Now that you have the repository info in place, you'll want to run
another update so that APT knows where to find the CloudStack packages.

.. code:: bash

    $ sudo apt-get update

You can now move on to the instructions under Install on Ubuntu.

Building RPMs from Source
-------------------------

As mentioned previously in `Section 3.3, “Prerequisites for building
Apache CloudStack” <#sect-source-prereq>`__, you will need to install
several prerequisites before you can build packages for CloudStack. Here
we'll assume you're working with a 64-bit build of CentOS or Red Hat
Enterprise Linux.

::

    # yum groupinstall "Development Tools"

::

    # yum install java-1.6.0-openjdk-devel.x86_64 genisoimage mysql mysql-server ws-commons-util MySQL-python tomcat6 createrepo

Next, you'll need to install build-time dependencies for CloudStack with
Maven. We're using Maven 3, so you'll want to `grab a Maven 3
tarball <http://maven.apache.org/download.cgi>`__ and uncompress it in
your home directory (or whatever location you prefer):

::

    $ tar zxvf apache-maven-3.0.4-bin.tar.gz

::

    $ export PATH=/usr/local/apache-maven-3.0.4//bin:$PATH

Maven also needs to know where Java is, and expects the JAVA\_HOME
environment variable to be set:

::

    $ export JAVA_HOME=/usr/lib/jvm/jre-1.6.0-openjdk.x86_64/

Verify that Maven is installed correctly:

::

    $ mvn --version

You probably want to ensure that your environment variables will survive
a logout/reboot. Be sure to update ``~/.bashrc`` with the PATH and
JAVA\_HOME variables.

Building RPMs for CloudStack is fairly simple. Assuming you already have
the source downloaded and have uncompressed the tarball into a local
directory, you're going to be able to generate packages in just a few
minutes.

.. note:: Packaging has Changed. If you've created packages for CloudStack previously, you should be
aware that the process has changed considerably since the project has moved to using Apache Maven. Please be sure to follow the steps in this section closely.

Generating RPMS
~~~~~~~~~~~~~~~

Now that we have the prerequisites and source, you will cd to the `packaging/centos63/` directory.

::

    $ cd packaging/centos63

Generating RPMs is done using the ``package.sh`` script:

::

    $./package.sh

That will run for a bit and then place the finished packages in
``dist/rpmbuild/RPMS/x86_64/``.

You should see the following RPMs in that directory:

::

    cloudstack-agent-4.2.0.el6.x86_64.rpm
    cloudstack-awsapi-4.2.0.el6.x86_64.rpm
    cloudstack-cli-4.2.0.el6.x86_64.rpm
    cloudstack-common-4.2.0.el6.x86_64.rpm
    cloudstack-docs-4.2.0.el6.x86_64.rpm
    cloudstack-management-4.2.0.el6.x86_64.rpm
    cloudstack-usage-4.2.0.el6.x86_64.rpm

Creating a yum repo
^^^^^^^^^^^^^^^^^^^

While RPMs is a useful packaging format - it's most easily consumed from
Yum repositories over a network. The next step is to create a Yum Repo
with the finished packages:

::

    $ mkdir -p ~/tmp/repo

::

    $ cp dist/rpmbuild/RPMS/x86_64/*rpm ~/tmp/repo/

::

    $ createrepo ~/tmp/repo

The files and directories within ``~/tmp/repo`` can now be uploaded to a
web server and serve as a yum repository.

Configuring your systems to use your new yum repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that your yum repository is populated with RPMs and metadata we need
to configure the machines that need to install CloudStack. Create a file
named ``/etc/yum.repos.d/cloudstack.repo`` with this information:

::

   [apache-cloudstack]
   name=Apache CloudStack
   baseurl=http://webserver.tld/path/to/repo
   enabled=1
   gpgcheck=0

Completing this step will allow you to easily install CloudStack on a
number of machines across the network.

Building Non-OSS
----------------

If you need support for the VMware, NetApp, F5, NetScaler, SRX, or any
other non-Open Source Software (nonoss) plugins, you'll need to download
a few components on your own and follow a slightly different procedure
to build from source.

Why Non-OSS?
------------

Some of the plugins supported by CloudStack cannot be distributed with
CloudStack for licensing reasons. In some cases, some of the required
libraries/JARs are under a proprietary license. In other cases, the
required libraries may be under a license that's not compatible with
`Apache's licensing guidelines for third-party
products <http://www.apache.org/legal/resolved.html#category-x>`__.

#. 

   To build the Non-OSS plugins, you'll need to have the requisite JARs
   installed under the ``deps`` directory.

   Because these modules require dependencies that can't be distributed
   with CloudStack you'll need to download them yourself. Links to the
   most recent dependencies are listed on the `*How to build on master
   branch* <https://cwiki.apache.org/CLOUDSTACK/how-to-build-on-master-branch.html>`__
   page on the wiki.

#. 

   You may also need to download
   `vhd-util <http://download.cloud.com.s3.amazonaws.com/tools/vhd-util>`__,
   which was removed due to licensing issues. You'll copy vhd-util to
   the ``scripts/vm/hypervisor/xenserver/`` directory.

#. 

   Once you have all the dependencies copied over, you'll be able to
   build CloudStack with the ``nonoss`` option:

::

    $ mvn clean
    $ mvn install -Dnonoss

#. 

   Once you've built CloudStack with the ``nonoss`` profile, you can
   package it using the `Section 3.6, “Building RPMs from
   Source” <#sect-source-buildrpm>`__ or `Section 3.5, “Building DEB
   packages” <#sect-source-builddebs>`__ instructions.




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
