# Installation

## Prerequisites

* Install [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html). Make sure your JAVA_HOME environment variable is pointed to the Java 8 installation location. Make sure that your PATH includes $JAVA_HOME.

* Install [Gradle](https://gradle.org/install/), and update your PATH environment variable as instructed.

## Developing with Local .jars

1. Create a new directory and navigate inside it.

2. Add the top-level [build.gradle](TODO) to this directory.

3. Add the [`libs`](libs) directory and its contents.

You can now import the `build.gradle` into your IDE of choice and start developing, as long as you conform to Gradle's [standard directory layout](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_project_layout).

## Building with Artifactory TODO

Add the following dependencies in your `build.gradle`

```groovy
dependencies {
  compile 'ai.swim:swim-client:0.1.0.20180216001614'
  compile 'ai.swim:swim-server:0.1.0.20180216001614'
}
```

and the following repository

```groovy
repositories {
  maven {
    url 'https://repo.swim.it/swim-releases/'
    credentials {
      username "${artifactoryUserName}"
      password "${artifactoryUserPassword}"
    }
    authentication {
      digest(BasicAuthentication)
    }
  }
}
```
To actually use the 

# Introduction
Swim is an eventually consistent, real-time, distributed object system. The building blocks of a SWIM server are `Services`, `Lanes`, `Links`, and a single `Plane`, where

* `Services` are objects
* `Lanes` are the fields and methods, of `Services`
* `Links` are references to `Lanes` in `Services`
* The `Plane` is a collection of `Service` definitions.

Public `Services` and `Lanes` form a Swim API (streaming API over web-sockets).

# Swim Concepts

## Services
Swim `Services` are distributed objects. A `Service` has the following features:
 
* Stores optionally-persistent state and data in `Lanes`
* Contains execution code (methods) defined in `Lane` call-back functions
* Instantiated lazily when a URI associated with a service/lane instance is invoked for the first time
* Is thread-safe
* Can dynamically change its type(s) and implement many types (dynamic polymorphism)
* Has a persistent universal address expressed as a URI
  
## Lanes
`Lanes` are members of `Services`. A `Lane` has the following features:

* Stores data and/or defines execution code
* Has a relative URI which is resolved into a fully qualified URI by pre-pending the URI of it's parent `Service`
* Accessible via `Links` from within a `Service` and outside a `Service`
* Persistent (i.e data is retained upon application/service restart) by default, but can be configured via the `transient` flag to keep data solely in memory
* Backed internally by [Recon](https://github.com/swimit/recon-java)  
* Is a parameterized class. Any class type can be specified provided there is a Recon transformation from/to that class.

The most prominent lane types are

1. `ValueLane`: Stores a single item and is accessed with a `ValueDownLink`. Developers can override call-back functions that execute when the `ValueLane` is updated or about to be updated.
2. `MapLane`: Stores a key-value map and is accessed with `MapDownlink`. Developers can override call-back functions that execute when the `MapLane` is updated or about to be updated.
3. `CommandLane`: A stateless lane for taking action and invoked with `commands`. Developers can override call-back functions that execute when the `CommandLane` is commanded. 
4. `JoinLane`: Aggregates multiple downlinks in a single lane. Automatically relinks to its aggregated lanes if a service restarts. (TODO: mention JoinMapLane vs JoinValueLane?)

The call-back functions for a given `Lane` can directly access data from other `Lanes` in the same `Service` instance. However, these functions can also read from, write to, and `command` public `Lanes` in _any Swim service, including those in a different_ `Plane`, by using the aforementioned Swim API.
              
## Links
A `Link` is a stateful read- and write-capable subscription to a `Lane` of some `Service` instance. A `Link` has the following features:

* Resilient to network congestion and failures 
* Multiplexed to handle multiple connections to different `Lanes` in a `Service` instance over a single connection

The main prominent `Link` types are

1. `ValueDownLink`: links to a `ValueLane` of a `Service` instance. Provides call-back functions to execute code when the `ValueLane` gets updated.
2. `MapDownLink`: links to a `MapLane` of a `Service` instance. Provides call-back functions to execute code when the `MapLane` gets updated.

## Planes
A `Plane` is used to start a Swim application. A `Plane` has the following features

* Has a collection of `Service` URI definitions
* Has application configuration, including but not limited to
  * http/https protocol parameters
  * port bindings
  * TLS parameters
  * `Lane` persistent storage directory
* Starts the Swim application
 
# Wiring it all together
These are the steps to build a Swim Application

1. Write Swim `Services` with the appropriate `Lane` definitions
2. Write a `Plane` with the `Service` URI definitions and the application configuration
3. Ingest data into `Lanes` using `commands` or `Links`. This can be done by a `SwimClient` (Java) or via an external program (non-Java).

That's it! `Services` spawn lazily when a URI associated with a `Service` or `Lane` instance is invoked for the first time, so Swim `Services` will process data as soon as it is available without requiring explicit instantiation.
    
For further reading 
Basics
Joins
