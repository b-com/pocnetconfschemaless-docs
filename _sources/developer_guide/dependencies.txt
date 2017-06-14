.. _poc-dependencies:

ODL Dependencies
================

This section lists the ODL dependencies we need to work with NETCONF devices
from the code. It shows how to declare those dependencies in the build
system xml files of our project.

impl
----

pom.xml
~~~~~~~

The project depends on the ``ietf-topology`` MD-SAL model
and on the ``sal-netconf-connector`` artifact of ODL netconf
project. In the ``impl/pom.xml`` file, add the following
dependency to the ``<dependencies>`` section:

.. code-block:: xml

   <dependency>
     <groupId>org.opendaylight.mdsal.model</groupId>
     <artifactId>ietf-topology</artifactId>
   </dependency>

   <dependency>
     <groupId>org.opendaylight.netconf</groupId>
     <artifactId>sal-netconf-connector</artifactId>
     <version>${mdsal.version}</version>
   </dependency>

.. note::

   For some reason, ``sal-netconf-connector`` uses the same version numbering
   as the mdsal.

.. _poc-dependencies-blueprint:

blueprint
~~~~~~~~~

In ``impl/src/main/resources/org/opendaylight/blueprint/impl-blueprint.xml``,
we need to declare that we want to use the DOM mount point service and the
DOM data broker. The former will allow us to get the NETCONF mount point of a
given device, and the latter will allow us to trigger NETCONF operations with
the devices. Excerpt:

.. code-block:: xml

   <reference id="domMountPointService"
              interface="org.opendaylight.controller.md.sal.dom.api.DOMMountPointService"/>

   <reference id="domDataBroker"
              interface="org.opendaylight.controller.md.sal.dom.api.DOMDataBroker"
              odl:type="default" />

   <bean id="provider"
     class="com.bcom.pocnetconfschemaless.impl.PocnetconfschemalessProvider"
     init-method="init" destroy-method="close">
     <argument ref="rpcRegistry" />
     <argument ref="domMountPointService" />
     <argument ref="domDataBroker" />
   </bean>

features
--------

pom.xml
~~~~~~~

In ``features/pom.xml``, we need to declare that we depend on
``features-netconf-connector``. The version number is not inherited from the
parent, so we need to give it. Excerpts:

.. code-block:: xml

   <properties>
     <!-- ... -->
     <netconf-connector.version>1.2.0-Carbon</netconf-connector.version>
   </properties>

.. code-block:: xml

   <dependencies>
     <!-- ... -->
     <dependency>
       <groupId>org.opendaylight.netconf</groupId>
       <artifactId>features-netconf-connector</artifactId>
       <version>${netconf-connector.version}</version>
       <classifier>features</classifier>
       <type>xml</type>
       <scope>runtime</scope>
     </dependency>
   </dependencies>

features.xml
~~~~~~~~~~~~

In ``features/src/main/features/features.xml``, we need to declare the
repository that contains the NETCONF features:

.. code-block:: xml

   <repository>mvn:org.opendaylight.netconf/features-netconf-connector/{{VERSION}}/xml/features</repository>

In that same file, we need to tell that our impl bundle depends on two
features of the netconf project: ``odl-netconf-connector`` and
``odl-netconf-topology``:

.. code-block:: xml
   :emphasize-lines: 3,4

   <feature name='odl-pocnetconfschemaless' version='${project.version}' description='OpenDaylight :: pocnetconfschemaless'>
     <feature version='${mdsal.version}'>odl-mdsal-broker</feature>
     <feature version='${netconf-connector.version}'>odl-netconf-connector</feature>
     <feature version='${netconf-connector.version}'>odl-netconf-topology</feature>
     <feature version='${project.version}'>odl-pocnetconfschemaless-api</feature>
     <bundle>mvn:com.bcom.pocnetconfschemaless/pocnetconfschemaless-impl/{{VERSION}}</bundle>
   </feature>

``odl-netconf-connector`` includes the ``sal-netconf-connector`` bundle.

``odl-netconf-topology`` allows us to mount and unmount NETCONF devices with
RESTConf.
