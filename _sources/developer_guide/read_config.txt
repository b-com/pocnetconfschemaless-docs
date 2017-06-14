.. _read-config:

Read the configuration of a mounted NETCONF device
==================================================

Build a read-only transaction
-----------------------------

Here we assume that we already have a mount point object, and that it is called
``mountPoint``. The steps needed to get this object are explained in
:ref:`get-mount-point`.

Building a read-only transaction is done in two steps:

1. Get a DOM data broker from the mount point.

2. Get the transaction from the data broker.

This is shown in the following code example:

.. code-block:: java

   final DOMDataBroker dataBroker = mountPoint.getService(DOMDataBroker.class).get();
   DOMDataReadOnlyTransaction rtx = dataBroker.newReadOnlyTransaction();

Build a YANG instance identifier
--------------------------------

NETCONF is able to filter the return of the ``get-config`` operation on
the device in order to retrieve only a subset of the configuration. This
capability reduces the amount of data exchanged between the client and the
device. Besides, it simplifies the work of the client: it is easier to look
for something in a smaller data set.

With ODL, NETCONF filtering can be accomplished by passing a YANG instance
identifier object to the ``read()`` method of the transaction. This identifier
can be seen as a path to the desired configuration subset.

Example: get the hostname of a JunOS device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the configuration of the Juniper router used in this poc, the path to the
hostname can be shown on the following XML code that only retains the
hostname and its containers and drops everything else:

.. code-block:: xml

   <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
     <system>
       <host-name>the-device-hostname</host-name>
     </system>
   </configuration>

The following Java code example shows how to build a YANG instance identifier
to retrieve only the desired configuration subset:

.. code-block:: java

   String JUNOS_NS = "http://xml.juniper.net/xnm/1.1/xnm";

   YangInstanceIdentifier yiid = YangInstanceIdentifier.builder()
       .node(new NodeIdentifier(QName.create(JUNOS_NS, "configuration")))
       .node(new NodeIdentifier(QName.create(JUNOS_NS, "system")))
       .node(new NodeIdentifier(QName.create(JUNOS_NS, "host-name")))
       .build();

Get the whole configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In case we want to retrieve the whole configuration, we must use the special
value ``YangInstanceIdentifier.EMPTY``.

Read the configuration
----------------------

To actually read the configuration, we call the ``read()`` method of the
transaction created earlier. We give it two parameters:

1. The desired logical datastore (here, the configuration datastore to get
   the running configuration of the device).

2. The YANG instance identifier created in the previous step.

To read the configuration synchronously, i.e. to be block until we get the
reply, we do:

.. code-block:: java

   Optional<NormalizedNode<?, ?>> opt = rtx.read(
       LogicalDatastoreType.CONFIGURATION, yiid).checkedGet();

If we get back to our Juniper example, ODL will send a NETCONF
message to the device that is similar to:

.. code-block:: xml

   <rpc message-id="m-0" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
     <get-config>
       <source>
         <running/>
       </source>
       <filter xmlns:ns0="urn:ietf:params:xml:ns:netconf:base:1.0" ns0:type="subtree">
         <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
           <system>
             <host-name/>
           </system>
         </configuration>
       </filter>
     </get-config>
   </rpc>

Look for the desired information in the output
----------------------------------------------

Assuming the device hostname is ``pjunren1``, the device will reply with
something similar to:

.. code-block:: xml

   <rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:junos="http://xml.juniper.net/junos/14.2R1/junos" message-id="m-0">
     <data>
       <configuration xmlns="http://xml.juniper.net/xnm/1.1/xnm">
         <system>
           <host-name>pjunren1</host-name>
         </system>
       </configuration>
     </data>
   </rpc-reply>

ODL will parse the reply, build a XML DOM object holding the contents of the
reply, and encapsulate the DOM object in a ``AnyXmlNode`` object.

To get the DOM object from the reply of the ``read()`` method of the
transaction, we can do:

.. code-block:: java

   if (opt.isPresent()) {
       final AnyXmlNode anyXmlData = (AnyXmlNode) opt.get();
       Node node = anyXmlData.getValue().getNode();
   }

Then we can use standard Java DOM techniques to extract the hostname from the
DOM object. For instance:

.. code-block:: java

   String hostname = null;

   Element root = ((Document) node).getDocumentElement();

   if (root.getNodeName().equals("data")) {
       NodeList hostnameNodes = root.getElementsByTagName("host-name");
       Node hostnameNode = hostnameNodes.item(0);
       NodeList childs = hostnameNode.getChildNodes();
       for (int i = 0; i < childs.getLength(); i++) {
           Node child = childs.item(i);
           if (child instanceof Text) {
               hostname = child.getNodeValue();
               break;
           }
       }
   }

In this example, if no text element with tag name ``host-name`` can be found,
the ``hostname`` variable will remain ``null``.

Check for possible errors
-------------------------

Using ODL in schemaless mode to read the configuration datastore of a NETCONF
device and looking for a value in the response can fail in three different ways:

1. ``ReadFailedException`` thrown by the transaction's ``read()`` method.

2. No data returned by the transaction's ``read()`` method.

3. Searched value not found in the data returned by the transaction's
   ``read()`` method.

Here is a code skeleton that shows where the different errors can be caught:

.. code-block:: java

   try {
       Optional<NormalizedNode<?, ?>> opt = rtx.read(
               LogicalDatastoreType.CONFIGURATION, yyid).checkedGet();

       if (opt.isPresent()) {
           final AnyXmlNode anyXmlData = (AnyXmlNode) opt.get();
           Node node = anyXmlData.getValue().getNode();
           hostname = lookForHostnameInNode(node);
           if (null == hostname) {
               /* error 3: hostname not found */
           }
       } else {
           /* error 2: no data returned */
       }
   } catch (ReadFailedException e) {
       /* error 1: exception while reading the datastore */
   } finally {
       rtx.close();
   }
