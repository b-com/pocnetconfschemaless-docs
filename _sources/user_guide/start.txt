Start pocnetconfschemaless
==========================

::

   $ cd ~/code/roads/pocnetconfschemaless/karaf/target/assembly
   $ bin/karaf
   opendaylight-user@root>log:set TRACE org.opendaylight.netconf.sal
   opendaylight-user@root>log:set TRACE com.bcom
   opendaylight-user@root>feature:install odl-pocnetconfschemaless-ui
   opendaylight-user@root>feature:install odl-netconf-console
   opendaylight-user@root>log:tail

ODL is ready when the logs show a line that looks like::

    2017-05-31 18:28:10,114 | INFO  | config-pusher    | ConfigPusherImpl                 | 131 - org.opendaylight.controller.config-persister-impl - 0.6.0.Carbon | Successfully pushed configuration snapshot 04-xsql.xml(odl-pocnetconfschemaless-ui,odl-pocnetconfschemaless-ui)

.. warning:: ``odl-pocnetconf-schemaless-*`` and ``odl-netconf-*`` features must
   not be installed automatically at the start of karaf, and this is the default
   behaviour of pocnetconfschemaless. Else, because of a bug, the SSH connection
   will not establish with some network devices such as netconf-testool or the
   Juniper MX5 router.

.. note:: It is not necessary to install ``odl-netconf-console`` to use the
   poc. However, it is useful to see the connection state of ODL and the mounted
   NETCONF devices.

.. note:: The ``log:set TRACE org.opendaylight.netconf.sal`` command allows to
   see the NETCONF message sent and received by ODL in ODL logs.
