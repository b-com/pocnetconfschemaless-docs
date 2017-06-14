Build pocnetconfschemaless
==========================

The present version of pocnetconfschemaless is based on OpenDaylight Carbon
release.

In order to use the poc, you'll need to patch ODL netconf project to fix a
blocking bug.

.. note::

   The following instructions mention directories for pocnetconfschemaless
   code and for ODL netconf code. These are just conventions used at b-com.
   You may place the code wherever you like.

Get pocnetconfschemaless code
-----------------------------

For b-com developers with access to the internal forge::

   $ mkdir -p ~/code/roads
   $ git clone ssh://gitolite@forge.b-com.com/roads/SANDBOX/pocnetconfschemaless.git

For other developers::

   $ mkdir -p ~/code/roads
   $ git clone https://github.com/b-com/pocnetconfschemaless.git

In the rest of the documentation, we'll assume that the source code can be
found in ``~/code/roads/pocnetconfschemaless``.

Build pocnetconfschemaless... a first time
------------------------------------------

You'll build pocnetconfschemaless a first time using the maven artifacts
provided by ODL. It will allow you to get all the needed dependencies and
cache them in ``~/.m2/repository``. But you'll get the buggy netconf artifacts::

   $ cd ~/code/roads/pocnetconfschemaless
   $ mvn clean install -DskipTests

.. note::

   pocnetconfschemaless tests usually pass at build time. However, they may
   fail in some environments, so you'll have better luck with ``-DskipTests``.

.. _patch-netconf:

Get, patch and build ODL netconf project
----------------------------------------

You'll get netconf code from ODL GitHub mirror and work on the
``release/carbon`` tag with::

   $ mkdir -p ~/code/opendaylight
   $ git clone https://github.com/opendaylight/netconf.git
   $ cd netconf
   $ git checkout release/carbon

Then you'll apply the bugfix patches with::

   $ cd ~/code/opendaylight/netconf
   $ git am < ~/code/roads/pocnetconfschemaless/0001-netconf-testtool-accept-RSA-hostname-key.patch
   Applying: netconf-testtool: accept RSA hostname key
   $ git am < ~/code/roads/pocnetconfschemaless/0002-fix-for-schemaless-devices.patch
   Applying: fix for schemaless devices

.. note::

   The patch ``0001-netconf-testtool-accept-RSA-hostname-key.patch`` is not
   strictly necessary. It permits to connect to ``netconf-testtool`` with
   Ubuntu 16.04 ``ssh`` client, which is sometimes useful but not necessary to
   use the poc. The patch works with Boron and Carbon.

.. note::

   The patch ``0002-fix-for-schemaless-devices.patch`` fixes the issue
   :ref:`issue-bad-transformer`. It will be required until the problem is fixed
   in ODL.

Finally, you'll build the patched netconf project and overwrite the
artifacts in ``~/.m2/repository`` with::

   $ cd ~/code/opendaylight/netconf
   $ mvn clean install -DskipTests

Re-build pocnetconfschemaless
-----------------------------

Now you'll rebuild pocnetconfschemaless using the newly-generated netconf
artifacts with::

   $ cd ~/code/roads/pocnetconfschemaless
   $ mvn clean install -DskipTests

You know have a usable karaf distribution in
``~/code/roads/pocnetconfschemaless/karaf/target/assembly``.

.. Reminder: in previous versions of the poc, we used snapshot versions
   during the Carbon development. So we needed to use the ``-nsu`` option
   (no snapshot update) to avoid the update of the cache with a more recent
   version of netconf artifacts.
