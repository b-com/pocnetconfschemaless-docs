Developer guide overview
========================

Scope
-----

How to interact programmatically from ODL with the configuration datastore of a
NETCONF device mounted with the ``schemaless`` flag.

Outline
-------

Dependencies
~~~~~~~~~~~~

* Declare the ODL dependencies we need at build time and run time. See
  :ref:`poc-dependencies`.

Read the configuration datastore
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Get the device mount point (assuming the device is already mounted). See
  :ref:`get-mount-point`.

* Build a read-only transaction

* Build a YANG instance identifier to limit the reading to a subset of
  the configuration XML tree (NETCONF filtering)

* Read the datastore

* Check for possible errors

* Look for the desired information in the output (DOM object)

For detailed explanations, see :ref:`read-config`.

Change the configuration datastore
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Get the device mount point (assuming the device is already mounted). See
  :ref:`get-mount-point`.

* Build a write-only transaction

* Build the XML for the desired configuration snippet, and encapsulate that
  in an ``AnyXmlNode`` object.

* Build a YANG instance identifier to point to the place where the
  configuration must be put or merged in the XML of the configuration datastore.

* Put or merge the transaction

* Submit the transaction

* Check for possible errors

For detailed explanations, see :ref:`edit-config`.
