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

Choosing a Hypervisor
=====================

CloudStack supports many popular hypervisors. Your cloud can consist
entirely of hosts running a single hypervisor, or you can use multiple
hypervisors. Each cluster of hosts must run the same hypervisor.

You might already have an installed base of nodes running a particular
hypervisor, in which case, your choice of hypervisor has already been
made. If you are starting from scratch, you need to decide what
hypervisor software best suits your needs. A discussion of the relative
advantages of each hypervisor is outside the scope of our documentation.
However, it will help you to know which features of each hypervisor are
supported by CloudStack. The following table provides this information.

======================================================================================================  ===============  ===============  ==============  ===========
Feature                                                                                                 XenServer 6.0.2  vSphere 4.1/5.0  KVM - RHEL 6.2  Bare Metal
======================================================================================================  ===============  ===============  ==============  ===========
Network Throttling                                                                                      Yes              Yes              No              N/A
Security groups in zones that use basic networking                                                      Yes              No               Yes             No
iSCSI                                                                                                   Yes              Yes              Yes             N/A
FibreChannel                                                                                            Yes              Yes              Yes             N/A
Local Disk                                                                                              Yes              Yes              Yes             Yes
HA                                                                                                      Yes              Yes (Native)     Yes             N/A
Snapshots of local disk                                                                                 Yes              Yes              Yes             N/A
Local disk as data disk                                                                                 No               No               No              N/A
Work load balancing                                                                                     No               DRS              No              N/A
Manual live migration of VMs from host to host                                                          Yes              Yes              Yes             N/A
Conserve management traffic IP address by using link local network to communicate with virtual router   Yes              No               Yes             N/A
======================================================================================================  ===============  ===============  ==============  ===========