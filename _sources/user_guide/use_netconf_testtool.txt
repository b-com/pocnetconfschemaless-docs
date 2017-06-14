Use pocnetconfschemaless with netconf-testtool
==============================================

Instead of a real network device, pocnetconfschemaless can work with the
NETCONF device simulator provided by ODL netconf project: netconf-testtool.

We use the poc by sending RESTConf requests:

* to ODL netconf configuration service (to mount the device in ODL),

* to the pocnetconfschemaless service (to read and write the device hostname).

In the examples, we assume that the RESTConf client (eg Firefox with the
RESTClient plugin) is on the same machine as ODL. We will connect to ODL
with the default credentials: login ``admin`` and the password ``admin``.

Start netconf-testtool
----------------------

For the needs of the poc, we wrote a small YANG module that describes the
structure of the configuration used by netconf-testtool. The contents of that
YANG module can be seen with::


    $ cat ~/code/roads/pocnetconfschemaless/netconf-testtool-config/yang-hostname-v2/hostname@2017-03-31.yang
    module hostname {
        yang-version 1;
        namespace "urn:opendaylight:hostname";
        prefix "hn";

        revision "2017-03-31";

        container system {
            leaf hostname {
                type string;
            }
            leaf location {
                type string;
            }
        }
    }

You'll start netconf-testtool with::

   $ cd ~/code/roads/pocnetconfschemaless/netconf-testtool-config
   $ java -jar ~/code/opendaylight/netconf/netconf/tools/netconf-testtool/target/netconf-testtool-1.2.0-Carbon-executable.jar \
       --schemas-dir yang-hostname-v2/  --debug true


.. _mount-netconf-testtool:

Mount the netconf-testtool device
---------------------------------

::

    PUT http://localhost:8181/restconf/config/network-topology:network-topology/topology/topology-netconf/node/netconf-testtool
    Content-Type: application/xml
    Accept: application/xml
    <node xmlns="urn:TBD:params:xml:ns:yang:network-topology">
      <node-id>netconf-testtool</node-id>
      <host xmlns="urn:opendaylight:netconf-node-topology">127.0.0.1</host>
      <port xmlns="urn:opendaylight:netconf-node-topology">17830</port>
      <username xmlns="urn:opendaylight:netconf-node-topology">admin</username>
      <password xmlns="urn:opendaylight:netconf-node-topology">admin</password>
      <tcp-only xmlns="urn:opendaylight:netconf-node-topology">false</tcp-only>
      <schemaless xmlns="urn:opendaylight:netconf-node-topology">true</schemaless>
    </node>

ODL replies with HTTP status code ``201 Created``.

The ``netconf:list-devices`` karaf command will allow you to confirm that the
netconf-testtool device is mounted and connected::

    opendaylight-user@root>netconf:list-devices
    NETCONF ID        | NETCONF IP | NETCONF Port | Status
    ---------------------------------------------------------
    netconf-testtool  | 127.0.0.1  | 17830        | connected
    controller-config | 127.0.0.1  | 1830         | connected

Read the hostname
-----------------

::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:get-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "netconf-testtool"
       }
   }

.. note::

   In all the calls to ``get-hostname`` et ``set-hostname``, we could add the
   NETCONF device family with ``"device-family": "netconf-testtool"``.

   Actually, this is not necessary because this is the default value.

Since the hostname is not defined yet, pocnetconftesttool replies with::

    {
         "output": {
             "hostname": "[noname4]"
         }
    }


Write the hostname
------------------

.. note::

   This test requires a fix in ODL netconf project (see
   :ref:`issue-bad-transformer`). The fix is already enabled if you followed
   the instructions in :ref:`patch-netconf`.

The following example shows how to set the hostname to the ``ntt`` value::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:set-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "netconf-testtool"
           "hostname": "ntt"
       }
   }

ODL replies with HTTP status code ``200 OK``.

In netconf-testtool logs and ODL logs, we can see (among others) the following
NETCONF message::

    <rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="m-4">
    <edit-config>
    <target>
    <candidate/>
    </target>
    <default-operation>none</default-operation>
    <config>
    <system xmlns="urn:opendaylight:hostname">
    <hostname xmlns:ns0="urn:ietf:params:xml:ns:netconf:base:1.0" ns0:operation="replace">ntt</hostname>
    </system>
    </config>
    </edit-config>
    </rpc>


Re-read the hostname
--------------------

As during the first read, we can do::

   POST http://localhost:8181/restconf/operations/pocnetconfschemaless:get-hostname
   Content-Type: application/yang.data+json
   Accept: application/json
   {
       "input": {
           "node-id": "netconf-testtool"
       }
   }

This time, pocnetconftesttool replies with the following contents::

    {
         "output": {
             "hostname": "ntt"
         }
    }

In netconf-testtool tools, we can see that ODL requests the device configuration
with the following NETCONF message::

    <rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="m-7">
    <get-config>
    <source>
    <running/>
    </source>
    </get-config>
    </rpc>

netconf-testtool replies with::

    <rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="m-7">
    <data>
    <system xmlns="urn:opendaylight:hostname">
    <hostname xmlns:ns0="urn:ietf:params:xml:ns:netconf:base:1.0" ns0:operation="replace">ntt</hostname>
    </system>
    </data>
    </rpc-reply>


Unmount the netconf-testtool device
-----------------------------------

When you have finished to work with the device, you can unmount it with the
following RESTConf request::

   DELETE http://localhost:8181/restconf/config/network-topology:network-topology/topology/topology-netconf/node/netconf-testtool
