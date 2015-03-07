Prepare the System VM Template
------------------------------

Secondary storage must be seeded with a template that is used for
CloudStack system VMs.

.. note::
   When copying and pasting a command, be sure the command has pasted as a 
   single line before executing. Some document viewers may introduce unwanted 
   line breaks in copied text.

#. On the Management Server, run one or more of the following
   ``cloud-install-sys-tmplt`` commands to retrieve and decompress the
   system VM template. Run the command for each hypervisor type that you
   expect end users to run in this Zone.

   If your secondary storage mount point is not named ``/mnt/secondary``,
   substitute your own mount point name.

   If you set the CloudStack database encryption type to "web" when you
   set up the database, you must now add the parameter ``-s
   <management-server-secret-key>``. See :ref:`about-password-key-encryption`.

   This process will require approximately 5 GB of free space on the
   local file system and up to 30 minutes each time it runs.

   *  For Hyper-V

      .. sourcecode:: bash

         /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
         -m /mnt/secondary \
         -u http://cloudstack.apt-get.eu/systemvm/4.5/systemvm64template-4.5-hyperv.vhd.zip \
         -h hyperv \
         -s <optional-management-server-secret-key> \
         -F

   *  For XenServer:

      .. sourcecode:: bash

         /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
         -m /mnt/secondary \
         -u http://cloudstack.apt-get.eu/systemvm/4.5/systemvm64template-4.5-xen.vhd.bz2 \
         -h xenserver \
         -s <optional-management-server-secret-key> \
         -F

   *  For vSphere:

      .. sourcecode:: bash

         /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
         -m /mnt/secondary \
         -u http://cloudstack.apt-get.eu/systemvm/4.5/systemvm64template-4.5-vmware.ova \
         -h vmware \
         -s <optional-management-server-secret-key> \
         -F

   *  For KVM:

      .. sourcecode:: bash

         /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
         -m /mnt/secondary \
         -u http://cloudstack.apt-get.eu/systemvm/4.5/systemvm64template-4.5-kvm.qcow2.bz2 \
         -h kvm \
         -s <optional-management-server-secret-key> \
         -F

   *  For LXC:

      .. sourcecode:: bash

         /usr/share/cloudstack-common/scripts/storage/secondary/cloud-install-sys-tmplt \
         -m /mnt/secondary \
         -u http://cloudstack.apt-get.eu/systemvm/4.5/systemvm64template-4.5-kvm.qcow2.bz2 \
         -h lxc \
         -s <optional-management-server-secret-key> \
         -F

#. If you are using a separate NFS server, perform this step. If you are
   using the Management Server as the NFS server, you MUST NOT perform
   this step.

   When the script has finished, unmount secondary storage and remove
   the created directory.

   .. sourcecode:: bash

      umount /mnt/secondary
      rmdir /mnt/secondary

#. Repeat these steps for each secondary storage server.
