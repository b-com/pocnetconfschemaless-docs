Use pocnetconfschemaless with a Nokia SR OS device
==================================================

In this example, we are working with a Nokia vSIM virtual router running SR
OS 14.0.R1. We assume that the initial router hostname is PE1, and we
will read and change that hostname.

We use the poc by sending RESTConf requests:

* to ODL netconf configuration service (to mount the device in ODL),

* to the pocnetconfschemaless service (to read and write the device hostname).

In the examples, we assume that the RESTConf client (eg Firefox with the
RESTClient plugin) is on the same machine as ODL. We will connect to ODL
with the default credentials: login ``admin`` and the password ``admin``.

Prerequisites
-------------

On the router:

* NETCONF must be enabled

* The ``admin`` user must be authorized to perform NETCONF locks

Mount the device
----------------

We choose to call the mount point ``vnokren1`` (shortcut for virtual Nokia
device located in Rennes number 1). The following RESTConf requests mounts
the device in ODL::

    PUT http://localhost:8181/restconf/config/network-topology:network-topology/topology/topology-netconf/node/vnokren1
    Content-Type: application/xml
    Accept: application/xml
    <node xmlns="urn:TBD:params:xml:ns:yang:network-topology">
      <node-id>vnokren1</node-id>
      <host xmlns="urn:opendaylight:netconf-node-topology">10.51.0.115</host>
      <port xmlns="urn:opendaylight:netconf-node-topology">830</port>
      <username xmlns="urn:opendaylight:netconf-node-topology">admin</username>
      <password xmlns="urn:opendaylight:netconf-node-topology">admin</password>
      <tcp-only xmlns="urn:opendaylight:netconf-node-topology">false</tcp-only>

      <schemaless xmlns="urn:opendaylight:netconf-node-topology">true</schemaless>

      <non-module-capabilities xmlns="urn:opendaylight:netconf-node-topology">
        <override>true</override>

        <capability>urn:ietf:params:netconf:base:1.0</capability>
        <capability>urn:ietf:params:netconf:base:1.1</capability>
        <capability>urn:ietf:params:netconf:capability:candidate:1.0</capability>
        <capability>urn:ietf:params:netconf:capability:validate:1.0</capability>
        <capability>urn:ietf:params:netconf:capability:validate:1.1</capability>
        <capability>urn:ietf:params:netconf:capability:startup:1.0</capability>
      </non-module-capabilities>
    </node>

.. note:: You will need to adapt the ``host``, ``username`` and ``password``
   fields to match your device configuration.

.. note::

   For a Nokia device, we use the ``<non-module-capabilities>`` container to
   override the capabilities of the device. This is to work around an issue
   with the NETCONF implementation of Nokia devices running SR OS.

   SR OS announces two NETCONF capabilities related to the way the
   NETCONF client must interact with the device configuration datastores:
   ``writable-running`` and ``candidate``. The former means that the running
   configuration (ie the current configuration) can be edited and will be
   effective immediately after edition. The latter
   means that the candidate configuration (ie a draft configuration) can be
   edited and will be validated with the ``commit`` operation.

   Normally, according to :rfc:`6241`, each datastore should have
   its own lock. But on Nokia devices, only one lock protects the two
   datastores. Taking one lock will take both locks. ODL is not aware of this
   proprietary behaviour: when editing the
   configuration, ODL will attempt to acquire both locks. The first attempt
   will succeed, but the second one will fail. In the end, the configuration
   change will not go well.

   Here, we override the device capabilities to only retain the ``candidate``
   capability. Consequently, ODL will only try to acquire the lock for the
   candidate configuration datastore.


ODL replies with HTTP code ``201 Created``.

The ``netconf:list-devices`` karaf command allows us to confirm that the
vnokren1 device is mounted::

    opendaylight-user@root>netconf:list-devices
    NETCONF ID        | NETCONF IP  | NETCONF Port | Status
    ----------------------------------------------------------
    vnokren1          | 10.51.0.115 | 830          | connected
    controller-config | 127.0.0.1   | 1830         | connected

Read the hostname
-----------------

::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:get-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "vnokren1",
           "device-family": "nokia"
       }
   }

pocnetconftesttool replies with the following contents::

    {
        "output":
        {
            "hostname" : "PE1"
        }
    }

Write (modify) the hostname
---------------------------

We will rename the device ``PE1-new`` with::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:set-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "vnokren1",
           "device-family": "nokia",
           "hostname": "PE1-new"
       }
   }

ODL replies with HTTP code ``200 OK``.

In ODL logs, we can see (among others) the following NETCONF message::

    <rpc message-id="m-3" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <edit-config>
    <target>
    <candidate/>
    </target>
    <config>
    <configure xmlns="urn:nokia.com:sros:ns:yang:sr:conf">
    <system xmlns="urn:nokia.com:sros:ns:yang:sr:conf-system">
    <name>PE1-new</name>
    </system>
    </configure>
    </config>
    </edit-config>
    </rpc>


Re-read hostname
----------------

As earlier, we ask for the hostname with::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:get-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "vnokren1",
           "device-family": "nokia"
       }
   }

This time, pocnetconftesttool replies with the new hostname::

    {
         "output": {
             "hostname": "PE1-new"
         }
    }

Unmount the device
------------------

When we have finished to work with the device, we can unmount it with the
following RESTConf request::

   DELETE http://localhost:8181/restconf/config/network-topology:network-topology/topology/topology-netconf/node/vnokren1
