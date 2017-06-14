.. _edit-config:

Edit the configuration of a mounted NETCONF device
==================================================

Overview
--------

We will illustrate how to edit the configuration of a mounted NETCONF device
by showing how to change the hostname of a Juniper NETCONF device.

On such a device, as already seen in the :ref:`read-config` section, the value
of the hostname can be found under the ``configuration/system/host-name``
tag hierarchy (or path):

.. code-block:: xml

   <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
     <system>
       <host-name>the-device-hostname</host-name>
     </system>
   </configuration>

In order to change the hostname to ``new-hostname``, we will:

1. Create a DOM document holding the ``<host-name>new-hostname</host-name>``
   configuration snippet, and encapsulate that document in a ``AnyXmlNode``
   object.

2. Create a YANG instance identifier holding the ``configuration/system/host-name``
   path that represents the insertion point of the configuration snippet.

3. Create a write-only transaction

4. Merge the ``AnyXmlNode`` object at the insertion point identified by
   the YANG instance identifier

5. Submit the transaction

Create the configuration snippet
--------------------------------

The following example shows how to create a DOM document holding the
``<host-name>new-hostname</host-name>`` configuration snippet using the
standard Java DOM APIs. This is a straightforward code that does not show
exception handling:

.. code-block:: java

   import javax.xml.parsers.DocumentBuilder;
   import javax.xml.parsers.DocumentBuilderFactory;
   import org.w3c.dom.Document;
   import org.w3c.dom.Element;

   // ...

   DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
   factory.setNamespaceAware(true);
   factory.setCoalescing(true);
   factory.setIgnoringElementContentWhitespace(true);
   factory.setIgnoringComments(true);

   DocumentBuilder builder = factory.newDocumentBuilder();

   // ...

   String JUNOS_NS = "http://xml.juniper.net/xnm/1.1/xnm";

   Document doc = builder.newDocument();
   Element hostnameElement = doc.createElementNS(JUNOS_NS, "host-name");
   doc.appendChild(hostnameElement);
   hostnameElement.appendChild(doc.createTextNode("new-hostname"));

Then we need to encapsulate that document in a ``AnyXmlNode`` object:

.. code-block:: java

   AnyXmlNode anyXmlNode = Builders.anyXmlBuilder()
       .withNodeIdentifier(new YangInstanceIdentifier.NodeIdentifier(
               QName.create(JUNOS_NS, "host-name"),
       .withValue(new DOMSource(doc.getDocumentElement()))
       .build();

Create the YANG instance identifier
-----------------------------------

We need to build a YANG instance identifier to point to the place where the
configuration must be merged in the device configuration datastore.

In our example, the YANG instance identifier must code the path
``configuration/system/host-name`` with the proper namespace. Here is a
code example that does just that:

.. code-block:: java

   YangInstanceIdentifier yiid = YangInstanceIdentifier.builder()
       .node(new NodeIdentifier(QName.create(JUNOS_NS, "configuration")))
       .node(new NodeIdentifier(QName.create(JUNOS_NS, "system")))
       .node(new NodeIdentifier(QName.create(JUNOS_NS, "host-name")))
       .build();

Build a write-only transaction
------------------------------

Here we assume that we already have a mount point object, and that it is called
``mountPoint``. The steps needed to get this object are explained in
:ref:`get-mount-point`.

Building a write-only transaction is done in two steps:

1. Get a DOM data broker from the mount point.

2. Get the transaction from the data broker.

This is shown in the following code example:

.. code-block:: java

   final DOMDataBroker dataBroker = mountPoint.getService(DOMDataBroker.class).get();
   DOMDataWriteTransaction writeTransaction = dataBroker.newWriteOnlyTransaction();

.. note::

   The act of creating a write-only transaction locks the datastore on the
   NETCONF device. ODL sends a message such as:

   .. code-block:: xml

      <rpc message-id="m-21" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <lock>
          <target>
            <candidate/>
          </target>
        </lock>
      </rpc>

Merge the configuration snippet
-------------------------------

To actually edit the configuration of the NETCONF device, we call
the ``merge()`` method of the write-only transaction
created earlier. We give it three parameters:

1. The desired logical datastore (here, the configuration datastore).

2. The YANG instance identifier created in the previous step.

3. The configuration snippet encapsulated in a ``AnyXmlNode``.


Example:

.. code-block:: java

   writeTransaction.merge(LogicalDatastoreType.CONFIGURATION, yiid, anyXmlNode);

In our example, ODL will send a NETCONF message to the device that is similar
to:

.. code-block:: xml

   <rpc message-id="m-22" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
     <edit-config>
       <target>
         <candidate/>
       </target>
       <config>
         <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
           <system>
             <host-name>new-hostname</host-name>
           </system>
         </configuration>
       </config>
     </edit-config>
   </rpc>

.. note::

   When we call the ``merge()`` method of the write-only transaction, ODL
   generates an ``edit-config`` NETCONF message whose ``operation`` attribute
   has the default value which is ``merge``. In this case:

   * everything found in the configuration snippet not found in the device
     configuration is created in the device configuration,

   * everything found in the device configuration but not found in the
     configuration snippet is left unchanged in the device configuration,

   * everything found in the configuration snippet and found the device
     configuration is replaced in the device configuration with the values in
     the configuration snippet.

   Alternatively, we could have used the ``put()`` method of the write-only
   transaction. In this case, ODL generates an ``edit-config`` NETCONF message
   where the ``operation`` attribute of the configuration snippet has the
   value ``replace``. In turn, the device should replace everything found
   at the location of the configuration snippet by the configuration snippet,
   potentially removing some configuration elements not found in the
   configuration snippet.

   Here is an example of a NETCONF message sent by ODL when we use the
   ``put()`` method:

   .. code-block:: xml

      <rpc message-id="m-32" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
          <target>
            <candidate/>
          </target>
          <default-operation>none</default-operation>
          <config>
            <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
              <system>
                <host-name xmlns:ns0="urn:ietf:params:xml:ns:netconf:base:1.0"
                           ns0:operation="replace">new-hostname</host-name>
              </system>
            </configuration>
          </config>
        </edit-config>
      </rpc>

   This mode does not work with our Juniper device, so we did not use it.

Submit the transaction
----------------------

In order to actually apply the new configuration on the device, we have to
call the ``submit()`` method of the write-only transaction. Example:

.. code-block:: java

   final CheckedFuture<Void, TransactionCommitFailedException> submit = writeTransaction.submit();

In our example, ODL will send a NETCONF commit message to the device such as:

.. code-block:: xml

   <rpc message-id="m-35" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
     <commit/>
   </rpc>

The ``submit()`` method is asynchronous.

In order to get a synchronous behaviour, we can do:

.. code-block:: java

   try {
      submit.checkedGet();
   } catch (TransactionCommitFailedException e) {
      /* handle exception */
   }

Alternatively, we can keep the asynchronous behaviour in our code and let ODL
block until completion after the return of our RPC implementation. In our
example, we do this by returning a ``Future<RpcResult<Void>>`` object with:

.. code-block:: java

   return Futures.transform(submit, new Function<Void, RpcResult<Void>>() {
       @Override
       public RpcResult<Void> apply(final Void result) {
          LOG.info("config writtent to '{}'", nodeId);
          return SUCCESS;
       }
   });

The problem with this alternate approach is that we do not know how to handle
gracefully exceptions that arise during the commit. For instance, if the device
is unmounted between the retrieval of the mount point and the call to
``submit()``, the user will get an obnoxious error.

.. note::

   At some point, ODL will release the lock on the device datastore. In ODL
   logs, we can see the sending of a NETCONF message such as:

   .. code-block:: xml

      <rpc message-id="m-36" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <unlock>
          <target>
            <candidate/>
          </target>
        </unlock>
      </rpc>

   With the ``checkedGet()`` approach, we observe that the message is sent
   during the execution of ``checkedGet()``.

   With the alternate approach, we observe that the message is sent after the
   ``apply()`` callback has been called.

Check for possible errors
-------------------------

Using ODL in schemaless mode to edit the configuration datastore of a NETCONF
device can fail in several different ways:

1. ``IllegalArgumentException`` may be thrown by the code that gets the mount
   point when the NETCONF device is not mounted in ODL.

2. ``IllegalStateException`` may be thrown at the retrieval of the data broker.

3. ``TransactionCommitFailedException`` may be thrown while waiting for the
   commit operation to complete.
