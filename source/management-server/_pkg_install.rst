Install the Management Server on the First Host
-----------------------------------------------

The first step in installation, whether you are installing the
Management Server on one host or many, is to install the software on a
single node.

.. note::
   If you are planning to install the Management Server on multiple nodes for 
   high availability, do not proceed to the additional nodes yet. That step 
   will come later.

The CloudStack Management server can be installed using either RPM or
DEB packages. These packages will depend on everything you need to run
the Management server.

.. include:: _pkg_repo.rst


Install on CentOS/RHEL
^^^^^^^^^^^^^^^^^^^^^^
   
.. sourcecode:: bash

   yum install cloudstack-management


Install on Ubuntu
^^^^^^^^^^^^^^^^^

.. sourcecode:: bash

   sudo apt-get install cloudstack-management

