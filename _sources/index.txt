.. pocnetconfschemaless documentation master file, created by
   sphinx-quickstart on Wed Apr  5 11:52:24 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

pocnetconfschemaless documentation
==================================

Overview
--------

pocnetconfschemaless is a proof of concept OpenDaylight application
that will show you how to read and write the configuration of a NETCONF device
that does not offer YANG models or whose YANG model cannot be parsed by ODL.

The poc will allow you to read and write the hostname of a variety of NETCONF
devices:

* netconf-testtool (NETCONF device simulator provided by ODL),

* Juniper MX5 routers,

* Nokia vSIM virtual routers.

pocnetconfschemaless has been developed at `b-com`_. It is licensed under the
`Eclipse Public License v1.0`_.

.. _b-com: https://b-com.com/
.. _Eclipse Public License v1.0: https://www.eclipse.org/legal/epl-v10.html

Reference
---------

The present documentation is available on `GitHub`_.

b-com developers can also find it on `b-com forge`_.

.. _GitHub: https://b-com.github.io/pocnetconfschemaless-docs/.
.. _b-com forge: https://forge.b-com.com/www/roads/pocnetconfschemaless/

User Guide
----------

.. toctree::
   :maxdepth: 2

   user_guide/build
   user_guide/start
   user_guide/use_netconf_testtool
   user_guide/use_junos_device
   user_guide/use_nokia_sros_device

Developer guide
---------------

.. toctree::

   developer_guide/overview
   developer_guide/dependencies
   developer_guide/mount_point
   developer_guide/read_config
   developer_guide/edit_config

OpenDaylight issues
-------------------

.. toctree::

   odl_issues/01_bad_transformer

Documentation developer guide
-----------------------------

.. toctree::

   developer_guide/build_the_docs

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
