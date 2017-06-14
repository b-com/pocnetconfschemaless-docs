.. _get-mount-point:

Get the mount point of a NETCONF device
=======================================

In order to perform transactions with a NETCONF device, ie in order to read
and write the datastores of a NETCONF device:

1. The NETCONF device must be mounted (see examples in the user guide, for
   instance :ref:`mount-netconf-testtool`),

2. We need to get the NETCONF device mount point.

We get the mount point using ODL mount point service.

We must declare that we want to use the mount point service in the bluprint
configuration file (see :ref:`poc-dependencies-blueprint`). This will make a
reference to a ``DOMMountPointService`` object available in the constructor of
``PocnetconfschemalessProvider``. We will pass that reference to the
constructor of the implementation of our service ``PocnetconfschemalessImpl`` for
further use.

To get the mount point of a given NETCONF device, we need to know
the device node ID that has been used at the creation of the mount point.

We use that ID (here ``nodeId``) to build a YANG instance identifier:

.. code-block:: java

    YangInstanceIdentifier mountPointId = YangInstanceIdentifier.builder()
            // network-topology:network-topology
            .node(NetworkTopology.QNAME)
            // .../topology/topology-netconf/
            .node(Topology.QNAME)
            .nodeWithKey(Topology.QNAME, QName.create(
                    Topology.QNAME, "topology-id").intern(),
                    TopologyNetconf.QNAME.getLocalName())
            // .../node/<nodeId>/
            .node(Node.QNAME)
            .nodeWithKey(Node.QNAME, QName.create(Node.QNAME, "node-id").intern(), nodeId)
            .build();

Using that instance identifier and the reference to the mount point service
(here ``domMountPointService``), we can get a reference to the mount point
of the desired NETCONF device:

.. code-block:: java

    final Optional<DOMMountPoint> nodeOptional = domMountPointService.getMountPoint(mountPointId);

    Preconditions.checkArgument(nodeOptional.isPresent(),
            "Unable to locate mountpoint: %s, not mounted yet or not configured", nodeId);

    DOMMountPoint mountPoint = nodeOptional.get();

.. note::

   This code throws a ``IllegalArgumentException`` exception if the device is
   not mounted or if the NETCONF topology does not exist.
