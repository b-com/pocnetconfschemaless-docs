.. _issue-bad-transformer:

ODL Issue 1: bad RPC structure transformer
==========================================

Overview
--------

In OpenDaylight netconf project, when a device is mounted with the
``schemaless`` option and when we try to edit its configuration,
NetconfRpcStructureTransformer is called
instead of SchemalessRpcStructureTransformer. ODL looks for the device schema
matching the namespace in the XML DOM that we built. This schema does not
exist, so ODL generates an exception.

I hit that bug in Boron. As of 2017-05-04, the bug is still present in Carbon.

The problem in detail
---------------------

Reproduce the problem
~~~~~~~~~~~~~~~~~~~~~

We want to send the following configuration snippet in a NETCONF edit-config
message to netconf-testtool::

   <system xmlns="urn:opendaylight:hostname">
   <hostname xmlns:ns0="urn:ietf:params:xml:ns:netconf:base:1.0" ns0:operation="replace">ntt</hostname>
   </system>

karaf log analysis
~~~~~~~~~~~~~~~~~~

karaf log shows the following exception::

    java.lang.NullPointerException: Cannot find (urn:opendaylight:hostname)system node in schema context. Instance identifier has to start from root
            at com.google.common.base.Preconditions.checkNotNull(Preconditions.java:250)[65:com.google.guava:18.0.0]
            at org.opendaylight.yangtools.yang.data.impl.schema.ImmutableNodes.fromInstanceId(ImmutableNodes.java:134)[81:org.opendaylight.yangtools.yang-data-impl:1.1.0.SNAPSHOT]
            at org.opendaylight.netconf.sal.connect.netconf.util.NetconfMessageTransformUtil.createEditConfigAnyxml(NetconfMessageTransformUtil.java:294)
            at org.opendaylight.netconf.sal.connect.netconf.util.NetconfRpcStructureTransformer.createEditConfigStructure(NetconfRpcStructureTransformer.java:39)
            at org.opendaylight.netconf.sal.connect.netconf.util.NetconfBaseOps.createEditConfigStrcture(NetconfBaseOps.java:257)
            at org.opendaylight.netconf.sal.connect.netconf.sal.tx.AbstractWriteTx.put(AbstractWriteTx.java:103)
            at com.bcom.pocnetconfschemaless.impl.Hostname.setHostname(Hostname.java:297)
            at com.bcom.pocnetconfschemaless.impl.PocnetconfschemalessImpl.setHostname(PocnetconfschemalessImpl.java:35)

It shows that netconf in ODL uses ``NetconfRpcStructureTransformer`` to
generate the XML that will be sent to the device. It looks for the schema
matching the namespace in the configuration snippet. It should not look for a
schema because we are in schemaless mode. Instead, it should use
``SchemalessRpcStructureTransformer`` that takes what it is given and does not
use any schema.

Problem correction
------------------

The correction consists in fixing the code run by ODL when it mounts a netconf
device and that is used to know whether the device must operate in schemaless
mode or not.

Here is the patch, developed for Boron and still applying for Carbon::

    diff --git a/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/sal/KeepaliveSalFacade.java b/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/sal/KeepaliveSalFacade.java
    index c450ac6..4026c38 100644
    --- a/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/sal/KeepaliveSalFacade.java
    +++ b/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/sal/KeepaliveSalFacade.java
    @@ -260,9 +260,9 @@ public final class KeepaliveSalFacade implements RemoteDeviceHandler<NetconfSess
          * DOMRpcService proxy that attaches reset-keepalive-task and schedule
          * request-timeout-task to each RPC invocation.
          */
    -    private static final class KeepaliveDOMRpcService implements DOMRpcService {
    +    public static final class KeepaliveDOMRpcService implements DOMRpcService {

    -        private final DOMRpcService deviceRpc;
    +        public final DOMRpcService deviceRpc;
             private ResetKeepalive resetKeepaliveTask;
             private final long defaultRequestTimeoutMillis;
             private final ScheduledExecutorService executor;
    diff --git a/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/util/NetconfBaseOps.java b/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/util/NetconfBaseOps.java
    index de61ee9..1b37015 100644
    --- a/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/util/NetconfBaseOps.java
    +++ b/netconf/sal-netconf-connector/src/main/java/org/opendaylight/netconf/sal/connect/netconf/util/NetconfBaseOps.java
    @@ -35,6 +35,7 @@ import com.google.common.util.concurrent.Futures;
     import com.google.common.util.concurrent.ListenableFuture;
     import org.opendaylight.controller.md.sal.dom.api.DOMRpcResult;
     import org.opendaylight.controller.md.sal.dom.api.DOMRpcService;
    +import org.opendaylight.netconf.sal.connect.netconf.sal.KeepaliveSalFacade;
     import org.opendaylight.netconf.sal.connect.netconf.sal.SchemalessNetconfDeviceRpc;
     import org.opendaylight.yang.gen.v1.urn.ietf.params.xml.ns.netconf.base._1._0.rev110601.copy.config.input.target.ConfigTarget;
     import org.opendaylight.yang.gen.v1.urn.ietf.params.xml.ns.netconf.base._1._0.rev110601.edit.config.input.EditContent;
    @@ -63,7 +64,15 @@ public final class NetconfBaseOps {
         public NetconfBaseOps(final DOMRpcService rpc, final SchemaContext schemaContext) {
             this.rpc = rpc;
             this.schemaContext = schemaContext;
    -        if (rpc instanceof SchemalessNetconfDeviceRpc) {
    +
    +        boolean schemaless = false;
    +        if (rpc instanceof KeepaliveSalFacade.KeepaliveDOMRpcService) {
    +            KeepaliveSalFacade.KeepaliveDOMRpcService rpc2 = (KeepaliveSalFacade.KeepaliveDOMRpcService) rpc;
    +            if (rpc2.deviceRpc instanceof SchemalessNetconfDeviceRpc) {
    +                schemaless = true;
    +            }
    +        }
    +        if (schemaless) {
                 this.transformer = new SchemalessRpcStructureTransformer();
             } else {
                 this.transformer = new NetconfRpcStructureTransformer(schemaContext);
