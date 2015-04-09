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

.. CloudStack Installation Documentation master file, created by
   sphinx-quickstart on Sat Jan 25 15:15:31 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


CloudStack Installation Documentation
=====================================

.. figure:: /_static/images/acslogo.png
   :align: center
   :target: http://cloudstack.apache.org/


This is the Apache CloudStack installation guide, for the Documentation home, 
the administrator guide or the Release-Notes please see:

-  `Documentation home`_

-  `Administration Guide`_

-  `Release Notes`_

.. note::
   In this guide we first go through some design and architectural choices_ to 
   build your cloud. Then we dive into a single node quick start guide_ to 
   give you a feel for the installation process. The source installation 
   steps_ are given in the follow-on section for people who want to build 
   their own packages. Otherwise you can use the general installation_ which 
   makes use of community maintained package repositories. The rest of the 
   guide goes through the configuration_ of the data-center and the setup of 
   the network_, storage_ and hypervisors_.

.. toctree::

.. _choices:


Choosing a Deployment Architecture
----------------------------------
.. toctree::
   :maxdepth: 2

   choosing_deployment_architecture

.. _guide:


Quick Installation Guide
------------------------
.. toctree::
   :maxdepth: 2

   qig


.. _steps:

Source Installation
-------------------
.. toctree::
   :maxdepth: 2

   building_from_source


.. _installation:

General Installation
--------------------
.. toctree::
   :maxdepth: 2

   overview/index
   management-server/index


.. _configuration:

Configuration
-------------
.. toctree::
   :maxdepth: 2

   configuration


.. _hypervisors:

Hypervisor Setup
----------------
.. toctree::
   :maxdepth: 2
   :glob:

   hypervisor/*

.. _network:


Network Setup
-------------
.. toctree::
   :maxdepth: 2

   network_setup

.. _storage:


Storage Setup
-------------
.. toctree::
   :maxdepth: 2

   storage_setup


Optional Installation 
---------------------
.. toctree::
   :maxdepth: 2

   optional_installation
   encryption

.. include:: _global.rst
