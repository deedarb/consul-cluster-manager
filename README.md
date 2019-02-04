# Consul cluster manager for Vert.x ecosystem #

[![Build Status](https://travis-ci.com/reactiverse/consul-cluster-manager.svg?branch=master)](https://travis-ci.com/reactiverse/consul-cluster-manager)

**Introduction**
-

IMPORTANT: This project has just got migrated from https://github.com/romalev/consul-cluster-manager and will get <b>ACTIVE</b> soon. Some things are still to be <b>UPDATED</b>!

Consul - based cluster manager that is plugable into Vert.x ecosystem. **[Consul](https://www.consul.io/)** is a distributed, highly available, and data center aware solution to connect and configure applications across dynamic, distributed infrastructure.

**Motivation**
- 
As we all know Vert.x cluster managers are pluggable and so far we have 6 cluster manager implementations: 

- Hazelcast - based: https://vertx.io/docs/vertx-hazelcast/java/
- Apache Zookeeper - based: https://vertx.io/docs/vertx-zookeeper/java/  
- Apache Ignite - based: https://vertx.io/docs/vertx-ignite/java/
- JGroups - based: https://github.com/vert-x3/vertx-jgroups
- Atomix - based : https://github.com/atomix/atomix-vertx
- Infinispan - based : https://github.com/vert-x3/vertx-infinispan 

If you are already using Consul along with Vert.x within your system - it might be a good fit to use Consul directly as main manager for : 
- discovery and group membership of Vert.x nodes in a cluster
Maintaining cluster wide topic subscriber lists (so we know which nodes are interested in which event bus addresses)
- distributed map support;
- distributed locks;
- distributed counters;   

Note : Cluster managers do not handle the event bus inter-node transport, this is done directly by Vert.x with TCP connections.

**Implementation details**
-
Current consul cluster manager implementation is fully based on [**vertx-consul-client**](https://vertx.io/docs/vertx-consul-client/java/) and [**vertx-core**](https://vertx.io/docs/vertx-core/java/).

**How to use**
-

### Gradle
```groovy

repositories {
    //...
    maven { url 'https://jitpack.io' }
}

compile 'io.reactiverse:vertx-consul-cluster-manager:${cluster-manager-version}'
```

### Maven
```xml

<project>
  <repositories>
    <repository>
      <id>jitpack</id>
      <url>https://jitpack.io</url>
    </repository>
  </repositories>
</project>

<dependency>
  <groupId>io.reactiverse</groupId>
  <artifactId>vertx-consul-cluster-manager</artifactId>
  <version>${cluster-manager-version}</version>
</dependency>
```
There's more than one way to create an instance of consul cluster manager. 
- Defaut one: 

``` ConsulClusterManager consulClusterManager = new ConsulClusterManager(); // Consul agent should be running then on localhost:8500.  ```
- Excplicilty specifying configuration: 
``` 
JsonObject options = new JsonObject()
.put("host", "localhost") // host on which consul agent is running, if not specified default host will be used which is "localhost".
.put("port", consulAgentPort) // port on wich consul agent is runing, if not specified default port will be used which is "8500".
/*
 * There's an option to utilize built-in internal caching. 
 * @{Code false} - enable internal caching of event bus subscribers - this will give us better latency but stale reads (stale subsribers) might appear.  
 * @{Code true} - disable internal caching of event bus subscribers - this will give us stronger consistency in terms of fetching event bus subscribers, 
 * but this will result in having much more round trips to consul kv store where event bus subs are being kept.
 */
.put("preferConsistency", false)
/*
 * There's also an option to specify explictly host address on which given cluster manager will be operating on. 
 * By defult InetAddress.getLocalHost().getHostAddress() will be executed.
 * Linux systems enumerate the loopback network interface the same way as regular LAN network interfaces, but the JDK       
 * InetAddress.getLocalHost method does not specify the algorithm used to select the address returned under such circumstances, and will 
 * often return the loopback address, which is not valid for network communication.
 */
 .put("nodeHost", "10.0.0.1");
 // consul client options can be additionally specified as needed.
ConsulClusterManager consulClusterManager = new ConsulClusterManager(options);
 ```
- Once cluster manager instance is created we can easily create clustered vertx. Voilà! 
```
VertxOptions vertxOptions = new VertxOptions();
vertxOptions.setClusterManager(consulClusterManager);
Vertx.clusteredVertx(vertxOptions, res -> {
    if (res.succeeded()) {
	    // clustered vertx instance has been successfully created!
	    Vertx vertx = res.result(); 
	} else {
	    // something went wrong :( 
	}
}
```

**Restrictions**
-
- Compliant with **ONLY Vert.x 3.6.0+** release.
- The limit on a key's value size of any of the consul  maps is 512KB.