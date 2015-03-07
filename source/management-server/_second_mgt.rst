Additional Management Servers
-----------------------------

For your second and subsequent Management Servers, you will install the
Management Server software, connect it to the database, and set up the
OS for the Management Server.

#. Perform the steps in `“Prepare the Operating System” 
   <#prepare-the-operating-system>`_ and `“Building RPMs from Source” 
   <building_from_source.html#building-rpms-from-source>`_ or 
   `“Building DEB packages” 
   <building_from_source.html#building-deb-packages>`_ as appropriate.

#. This step is required only for installations where XenServer is
   installed on the hypervisor hosts.

   Download vhd-util from
   `vhd-util <http://download.cloud.com.s3.amazonaws.com/tools/vhd-util>`_

   Copy vhd-util to
   ``/usr/share/cloudstack-common/scripts/vm/hypervisor/xenserver``.

#. Ensure that necessary services are started and set to start on boot.

   .. sourcecode:: bash

      service rpcbind start
      service nfs start
      chkconfig nfs on
      chkconfig rpcbind on

#. Configure the database client. Note the absence of the --deploy-as
   argument in this case. (For more details about the arguments to this
   command, see :ref:`install-database-on-separate-node`.)

   .. sourcecode:: bash

      cloudstack-setup-databases cloud:dbpassword@dbhost \
      -e encryption_type \
      -m management_server_key \
      -k database_key \
      -i management_server_ip

#. Configure the OS and start the Management Server:

   .. sourcecode:: bash

      cloudstack-setup-management

   The Management Server on this node should now be running.

#. Repeat these steps on each additional Management Server.

#. Be sure to configure a load balancer for the Management Servers. See
   `“Management Server Load Balancing” 
   <http://docs.cloudstack.apache.org/en/latest/administration_guide.html?highlight=management%20server%20load#management-server-load-balancing>`_.
