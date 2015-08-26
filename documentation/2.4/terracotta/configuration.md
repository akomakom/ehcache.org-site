---
---
# Distributed Cache Configuration


 


## Introduction

Terracotta configuration in ehcache.xml is done in three parts:

* CacheManager Configuration
* Terracotta Server Configuration
* Per cache configuration (enabling and settings)


## CacheManager Configuration


### Via ehcache.xml

The attributes of `<ehcache>` are:

* name &ndash;
an optional name for the CacheManager.  The name is optional and primarily used
for documentation or to distinguish Terracotta clustered cache state.  With Terracotta
clustered caches, a combination of CacheManager name and cache name uniquely identify a
particular cache store in the Terracotta clustered memory.
The name will show up in the Developer Console.
<a name="updateCheck"/>
* updateCheck &ndash;
an optional boolean flag specifying whether this CacheManager should check
for new versions of Ehcache over the Internet.  If not specified, updateCheck="true".
* monitoring &ndash;
an optional setting that determines whether the CacheManager should
automatically register the SampledCacheMBean with the system MBean server.  Currently,
this monitoring is only useful when using Terracotta and thus the "autodetect" value
will detect the presence of Terracotta and register the MBean.  Other allowed values
are "on" and "off".  The default is "autodetect".

        <Ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="ehcache.xsd"
        updateCheck="true" monitoring="autodetect">


### Programmatic Configuration

CacheManagers can be configured programmatically with a fluent API. The example below
creates a CacheManager with a Terracotta config specified in an URL, and creates a defaultCache
and a cache named "example".

    Configuration configuration = new Configuration()
    .terracotta(new TerracottaClientConfiguration().url("localhost:9510"))
    .defaultCache(new CacheConfiguration("defaultCache", 100))
    .cache(new CacheConfiguration("example", 100)
    .timeToIdleSeconds(5)
    .timeToLiveSeconds(120)
    .terracotta(new TerracottaConfiguration()));
    CacheManager manager = new CacheManager(configuration);


## Terracotta Server Configuration
Note: You need to install and run one or more Terracotta servers to use Terracotta clustering.
See the [download page](http://www.terracotta.org/dl).
With a server/servers up and running you need to specify the location of the servers.
Configuration can be specified in two main ways: by reference to a source of
configuration or by use of an embedded Terracotta configuration file.

### Specification of a source of configuration
To specify a reference to a source (or sources) of configuration, use the url
attribute.  The url attribute must contain a comma-separated list using the following forms::

* path to the Terracotta configuration file (tc-config.xml by default)

    Example using a path to Terracotta configuration file:

        <terracottaConfig url="/app/config/tc-config.xml"/>

* URL to the Terracotta configuration file

    Example using a URL to a Terracotta configuration file:

        <terracottaConfig url="http://internal/ehcache/app/tc-config.xml"/>

* &lt;server host>:&lt;port> of a running Terracotta Server instance

    Example pointing to a Terracotta server installed on localhost:

        <terracottaConfig url="localhost:9510"/>

    Example using multiple Terracotta server instance URLs (for fault tolerance):

        <terracottaConfig url="host1:9510,host2:9510,host3:9510"/>

### Specification using embedded tc-config
To embed a Terracotta configuration file within the Ehcache configuration, simply
place the usual Terracotta XML config within the `<terracottaConfig>` element.
In this example we have two Terracotta servers running on `server1` and `server2`.

    <terracottaConfig>
       <tc-config>
           <servers>
               <server host="server1" name="s1"/>
               <server host="server2" name="s2"/>
           </servers>
           <clients>
               <logs>app/logs-%i</logs>
           </clients>
       </tc-config>
    </terracottaConfig>

## Per Cache Configuration
From Ehcache 2.5, there are a couple of configuration attributes that only take effect on a Terracotta clustered cache:

*  `localCacheEnabled`=<true> | false - whether to store frequently used data, the hotset, in-process
   in the Ehcache node ("L1"). The default is true, which gives the usually required two tiered L1, L2 architecture.
   Setting this to false will not use the L1. All requests will go to Terracotta Server Array (the L2).

Individual caches are not clustered with Terracotta unless enabled to do so using the `<terracotta>` sub-element.
 The `<terracotta>` sub-element has the following attributes:

* clustered - Enables ("true" DEFAULT) or disables ("false") Terracotta clustering of a specific cache. Clustering is enabled if this attribute is not specified in the `<terracotta>` sub-element.
* consistency - Enables strong consistency ("strong" DEFAULT) or eventual consistency ("eventual"). When strong, guarantees the consistency of the cache’s data across the cluster at all times at the cost of performance. After any update is completed, no read can return a stale value. When eventual, improves performance while guaranteeing eventual cluster-wide cache consistency. Once set, this consistency mode cannot be changed except by reconfiguring the cache using a configuration file and reloading the file. This setting cannot be changed programmatically.
* synchronousWrites - Enables ("true") or disables ("false" DEFAULT) synchronous writes from Terracotta clients (application servers) to Terracotta servers. Asynchronous writes (synchronousWrites="false") maximize performance by allowing clients to proceed without waiting for a "transaction received" acknowledgement from the server. Synchronous writes (synchronousWrites="true") maximize data safety by requiring that a client receive server acknowledgement of a transaction before that client can proceed. If the cache’s consistency mode is eventual (consistency="eventual"), or while it is set to bulk load using the bulk-load API, only asynchronous writes can occur (synchronousWrites="true" is ignored).
* storageStrategy - Sets the strategy for storing the cache’s key set. Use "DCV2" (DEFAULT) to store the cache’s key set on the Terracotta server array. DCV2 can be used only with serializable caches (the valueMode attribute must be set to "serialization"), whether using the standard installation or DSO. Use "classic" to store all keys on every Terracotta client. Identity caches (valueMode="identity") must use the classic mode. For more information on using storageStrategy, see Offloading Large Caches.
* concurrency - Sets the number of segments for the map backing the underlying server store managed by the Terracotta Server Array. If concurrency is not explicitly set (or set to "0"), the system selects an optimized value. See [Tuning Concurrency](http://terracotta.org/documentation/enterprise-ehcache/reference-guide#86549) for more information on how to tune this value for DCV2.
* valueMode - Sets the type of cache to serialization (DEFAULT, the standard Ehcache "copy" cache) or identity (Terracotta object identity). Identity mode is not available with the standard (express) installation . Identity mode can be used only with a Terracotta DSO (custom) installation (see Standard Versus DSO Installations).
  In serialization mode, getting an element from the cache gets a copy of that element. Changes made to that copy do not affect any other copies of the same element or the value in the cache. Putting the element in the cache overwrites the existing value. This type of cache provides high performance with small, read-only data sets. Large data sets with high traffic, or caches with very large elements, can suffer performance degradation because this type of cache serializes clustered objects. This type of cache cannot guarantee a consistent view of object values in read-write data sets if the consistency attribute is disabled. Objects clustered in this mode must be serializable. Note that getKeys() methods return serialized versions of the keys.
  In identity mode, getting an element from the cache gets a reference to that element. Changes made to the referenced element updates the element on every node on which it exists (or a reference to it exists) as well as updating the value in the cache. Putting the element in the cache does not overwrite the existing value. This mode guarantees data consistency. It can be used only with a custom Terracotta Distributed Cache installation. Objects clustered in this mode must be portable and must be locked when accessed. If you require identity mode, you must use DSO (see Terracotta DSO Installation).
* copyOnRead – DEPRECATED. Use the copyOnRead `<cache>` attribute. Enables ("true") or disables ("false" DEFAULT) "copy cache" mode. If disabled, cache values are not deserialized on every read. For example, repeated get() calls return a reference to the same object (references are ==).
  When enabled, cache values are deserialized (copied) on every read and the materialized values are not re-used between get() calls; each get() call returns a unique reference. When enabled, allows Ehcache to behave as a component of OSGI, allows a cache to be shared by callers with different classloaders, and prevents local drift if keys/values are mutated locally without being put back into the cache. Enabling copyOnRead is relevant only for caches with valueMode set to serialization .
* coherentReads – DEPRECATED. This attribute is superseded by the attribute consistency . Disallows ("true" DEFAULT) or allows ("false") "dirty" reads in the cluster. If set to "true", reads must be consistent on any node and returned data is guaranteed to be consistent. If set to false, local unlocked reads are allowed and returned data may be stale. Allowing dirty reads may boost the cluster’s performance by reducing the overhead associated with locking. Read-only applications, applications where stale data is acceptable, and certain read-mostly applications may be suited to allowing dirty reads.

The `<terracotta>` sub-element also has a `<nonstop>` sub-element to allow configuration of cache behavior if a distributed
cache operation cannot be completed within a set time or in the event of a clusterOffline message. If the `<nonstop>` element is absent, this behavior is off.
`<nonstop>` has the following attributes:

*  enabled="true" - defaults to true.
*  timeoutMillis - An SLA setting, so that if a cache operation takes longer than the allowed ms, it will timeout.
*  immediateTimeout="true|false" - What to do on receipt of a ClusterOffline event indicating that communications  with the Terracotta Server Array were interrupted.
*  timeoutBehavior="noop|exception|localReads" - What to do when a timeout has occurred. See [NonStop Operation](./non-stop-cache) for more information on the use of nonstop.

The simplest way to enable clustering is to add:

    <terracotta/>

To indicate the cache should not be clustered (or remove the `<terracotta>` element altogether):

    <terracotta clustered="false"/>

To indicate the cache should be clustered using "eventual" consistency mode for better performance:

    <terracotta clustered="true" consistency="eventual"/>

To indicate the cache should be clustered using synchronous-write locking level:

    <terracotta clustered="true" synchronousWrites="true"/>

To indicate the cache should be clustered using identity mode:

    <terracotta clustered="true" valueMode="identity"/>

Following is an example Terracotta clustered cache named sampleTerracottaCache.

    <cache name="sampleTerracottaCache"
          maxElementsInMemory="1000"
          eternal="false"
          timeToIdleSeconds="3600"
          timeToLiveSeconds="1800"
          overflowToDisk="false">
       <terracotta/>
    </cache>

## Further Configuration Topics


### Copy On Read

The `copyOnRead` setting is most easily explained by first examining what it does when
not enabled and exploring the potential problems that can arise.
For a cache for which `copyOnRead` is NOT enabled, the following reference comparsion
will always be true (NOTE: assuming no other thread changes the cache mapping between the get()'s)

    Object obj1 = c.get("key").getValue();
    Object obj2 = c.get("key").getValue();
    if (obj1 == obj2) {
     System.err.println("Same value objects!");
    "/>

The fact that the same object reference is returned accross multiple get()'s implies that
the cache is storing a direct reference to cache value. When `copyOnRead` is enabled
the object references will be fresh and unique upon every get().
This default behavior (copyOnRead=false) is usually what you want although there are at least
two scenarios in which this is problematic:

(1) Caches shared between classloaders and

(2) Mutable value objects

Imagine two web applications that both have access to the same Cache instance (this implies that
the core ehcache classes are in a common classloader). Imagine further that the classes for value types
in the cache are duplicated in the web application (ie. they are not present in the common loader).
In this scenario you would get ClassCastExceptions when one web application accessed a value previously
read by the other application.

One obvious solution to this problem is to move the value types to the
common loader, but another is to enable `copyOnRead` so that thread context loader of the caller
will be used to materialize the cache values on each get(). This feature has utility in OSGi environments
as well where a common cache service might be shared between bundles
Another subtle issue concerns mutable value objects in a clustered cache. Consider this simple code
which shows a Cache that contains a mutable value type (Foo):

    class Foo {
     int field;
    "/>
    Foo foo = (Foo) c.get("key").getValue();
    foo.field++;
    // foo instance is never re-put() to the cache
    // ...

If the Foo instance is never re-put() to the Cache your local cache is no longer consistent with
the cluster (it is locally modified only). Enabling `copyOnRead` eliminates this possibility
since the only way to affect cache values is to call mutator methods on the Cache.
It is worth noting that there is a performance penalty to copyOnRead since values are deserialized
on every get().


### Cache listeners

Cache listeners listen for changes, including replicated cluster changes, made through the Cache API. Because
Terracotta cluster changes happen transparently directly to the `MemoryStore` a listener will not be invoked
when an event occurs out on the cluster. If it occurs locally, then it must have occurred through the Cache API, so
a local event will be detected by a local listener.
A common use of listeners is to trigger a reload of a just invalidated `Element`. In Terracotta clustering
this is avoided as a change in one node is always coherent to the other nodes.
To send out cache change events across the cluster, you need to set up a local listener on the relevant cache so
that these events can be propagated to other nodes. This is done by adding the following
`cacheEventListenerFactory` tag to the cache:

    <cacheEventListenerFactory
      class="net.sf.ehcache.event.TerracottaCacheEventReplicationFactory"/>


### Reconnect

In the event a socket closes due to a network error, if reconnect is enabled, the L1 will attempt to reconnect to the L2.
By default reconnect is turned off and a disconnected L1 will stay disconnected. This is generally not what you want.
Set reconnect to true in tcconfig to allow automatic reconnection.

    l2.l1reconnect.enabled=true

The tcconfig property `l2.l1reconnect.timeout.millis` determines how long an L1 will be allowed to attempt to reconnect.
By default this is 2 seconds.
 The L2s maintain a UUID of the L1s that were connected and have disconnected in a state table. If the timeout is exceeded
the L1 is prevented from reconnecting forever.
While reconnection is being attempted cache operations will be controlled by the `nonstop` configuration. By default this
is not enabled. The default behaviour is therefore for all cache operations to block until reconnect is established.
NonStop enables cache operations during reconnect.  See [NonStop Operation](./non-stop-cache) for more information
on the use of nonstop.
When a cache reconnects, it maintains it's local data and is considered to have the same UUID. Any locks held by it which have not
timed out are still in place.
It is recommended that rejoin, which is more comprehensive be used instead of reconnect.
Reconnect is still appropriate where you have a dirty network with lots of short blips. Reconnect will smooth that out, and provided
no blip lasts too long, will give you smooth response times.

### Rejoin
Rejoin is a comprehensive new capability added to Ehcache 2.4.1/Terracotta 3.5 to deal with disconnections between L1s and L2s.
This is not the default and must be enabled. Prior to this version the only way an Ehcache instance
can rejoin a cluster is by restarting the application server the Ehcache instance is in. This is still the case unless you enable
rejoin.
Regardless of where a connection between an L1 and L2 is terminated due:

*  a socket close due to a network disconnect
*  health checker evicting an L1 due to lack of responsiveness
*  reconnect fails because either reconnect was not enabled or the timeout has been exceeded


#### While Disconnected

Rejoin is used with nonstop. While the L1 is up but disconnected cache behaviour is controlled by the nonstop configuration.
See [NonStop Operation](/documentation/user-guide/non-stop-cache) for more information. In essence cache operations can be configured not to
block based on a CAP tradeoff you determine.


#### After Rejoin

On rejoin, the old L1 is discarded. A new L1 is created under Ehcache which will initially be empty, but will be connected to the
L2 and will pull through from the server cache entries on demand.
From the point of view of the L2s it is a brand new L1.  The consequences of this are:

*   the L1 gets a new UUID
*   any locks held are released
*   any local changes made to the cahce are discarded.

All of this is transparent to any code interacting with Ehcache in the user application.


#### Configuration

Rejoin must be configured at a `CacheManager` level. It thus applies to all Terracotta clustered caches within that CacheManager.
Rejoin can only be used in conjunction with NonStop. The nonstop sub-element must be econfigured.

    <ehcache name="cache4">
    ...
    <terracottaConfig rejoin="true" url="localhost:9510" />
    </ehcache>

## Further Information

See [Ehcache Configuration](http://www.terracotta.org/documentation/enterprise-ehcache/configuration-guide) in the Terracotta documentation for more information.