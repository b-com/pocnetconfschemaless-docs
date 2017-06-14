Use pocnetconfschemaless with a Juniper device running JunOS
============================================================

In our lab at b-com, we worked with a Juniper MX5 router running JunOS 14.2R1.9.
However, the poc should work with other Juniper devices running JunOS.

We assume that the initial router hostname is pjunren1, and we will read and
change that hostname.

We use the poc by sending RESTConf requests:

* to ODL netconf configuration service (to mount the device in ODL),

* to the pocnetconfschemaless service (to read and write the device hostname).

In the examples, we assume that the RESTConf client (eg Firefox with the
RESTClient plugin) is on the same machine as ODL. We will connect to ODL
with the default credentials: login ``admin`` and the password ``admin``.

Mount the device
----------------

We choose to call the mount point ``pjunren1`` (shortcut for physical Juniper
device located in Rennes number 1). The following RESTConf requests mounts
the device in ODL::

    PUT http://localhost:8181/restconf/config/network-topology:network-topology/topology/topology-netconf/node/pjunren1
    Content-Type: application/xml
    Accept: application/xml
    <node xmlns="urn:TBD:params:xml:ns:yang:network-topology">
      <node-id>pjunren1</node-id>
      <host xmlns="urn:opendaylight:netconf-node-topology">10.51.0.102</host>
      <port xmlns="urn:opendaylight:netconf-node-topology">830</port>
      <username xmlns="urn:opendaylight:netconf-node-topology">admin</username>
      <password xmlns="urn:opendaylight:netconf-node-topology">juniper1</password>
      <tcp-only xmlns="urn:opendaylight:netconf-node-topology">false</tcp-only>
      <schemaless xmlns="urn:opendaylight:netconf-node-topology">true</schemaless>
    </node>

.. note:: You will need to adapt the ``host``, ``username`` and ``password``
   fields to match your device configuration.

ODL replies with HTTP status code ``201 Created``.

The ``netconf:list-devices`` karaf command will allow you to check that the
pjunren1 device is mounted and connected::

    opendaylight-user@root>netconf:list-devices
    NETCONF ID        | NETCONF IP  | NETCONF Port | Status
    ----------------------------------------------------------
    controller-config | 127.0.0.1   | 1830         | connected
    pjunren1          | 10.51.0.102 | 830          | connected

Read the hostname
-----------------

::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:get-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "pjunren1",
           "device-family": "junos"
       }
   }

pocnetconftesttool replies with the following contents::

    {
         "output": {
             "hostname": "pjunren1"
         }
    }


Change the hostname
-------------------

In the following example, we change the hostname to the new value
``pjunren1new``::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:set-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "pjunren1",
           "device-family": "junos",
           "hostname": "pjunren1new"
       }
   }

ODL replies with HTTP status code ``200 OK``.

In ODL logs, we can see (among others) the following NETCONF message::

    <rpc message-id="m-29" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <edit-config>
    <target>
    <candidate/>
    </target>
    <config>
    <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
    <system>
    <host-name>pjunren1new</host-name>
    </system>
    </configuration>
    </config>
    </edit-config>
    </rpc>

Re-read the hostname
--------------------

As during the first read, we ask for the hostname with::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:get-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "pjunren1",
           "device-family": "junos"
       }
   }

pocnetconftesttool replies with the new contents::

    {
         "output": {
             "hostname": "pjunren1new"
         }
    }

Unmount the device
------------------

When you have finished to work with the device, you can unmount it with the
following RESTConf request::

   DELETE http://localhost:8181/restconf/config/network-topology:network-topology/topology/topology-netconf/node/pjunren1
