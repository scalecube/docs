# scalecube-services
[![Build Status](https://travis-ci.org/scalecube/scalecube-services.svg?branch=develop)](https://travis-ci.org/scalecube/scalecube-services)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.scalecube/scalecube-services-api/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.scalecube/scalecube-services-api)
[![SourceSpy Dashboard](https://sourcespy.com/shield.svg)](https://sourcespy.com/github/scalecubescalecubeservices/)

## MICROSERVICES 2.0

<table text-align="top">
 <tr>
   <td>
    An open-source project that is focused on streamlining reactive-programming of Microservices Reactive-systems that scale, built by developers for developers.<br><br>
ScaleCube Services provides a low latency Reactive Microservices library for peer-to-peer service registry and discovery 
based on gossip protocol, without single point-of-failure or bottlenecks.<br><br>
    Scalecube more gracefully address the cross cutting concernes of distributed microservices architecture.
    <br><br>
  </td>
  <td>
  <img src="https://user-images.githubusercontent.com/1706296/43058327-b4a0147e-8e4f-11e8-9999-68c4ec99632e.gif">
  </td>
</tr>
</table>
ScaleCube Services Features:

* Provision and interconnect microservices peers in a cluster
* Fully Distributed with No single-point-of-failure or single-point-of-bottleneck
* Fast - Low latency and high throughput
* Scaleable over- cores, jvms, clusters, regions.
* Built-in Service Discovery and service routing
* Zero configuration, automatic peer-to-peer service discovery using gossip
* Simple non-blocking, asynchronous programming model
* Reactive Streams support.
  * Fire And Forget - Send and not wait for a reply
  * Request Response - Send single request and expect single reply
  * Request Stream - Send single request and expect stream of responses. 
  * Request bidirectional - send stream of requests and expect stream of responses.
* Built-in failure detection, fault tolerance, and elasticity
* Routing and balancing strategies for both stateless and stateful services
* Embeddable into existing applications
* Natural Circuit-Breaker via scalecube-cluster discovery and failure detector.
* Support Service instance tagging.
* Modular, flexible deployment models and topology
* pluggable api-gateway providers (http / websocket / rsocket)
* pluggable service transports (tcp / aeron / rsocket)
* pluggable encoders (json, SBE, Google protocol buffers)

User Guide:

* [Services Overview](http://scalecube.io/services.html)
* [Defining Services](http://scalecube.io/user-reference/services/DefineService.html)
* [Implementing services](http://scalecube.io/user-reference/services/ServiceImplementation.html)
* [Provisioning Clustered Services](http://scalecube.io/user-reference/services/ProvisionClusterServices.html)
* [Consuming services](http://scalecube.io/user-reference/services/ConsumingServices.html)


Basic Usage:

The example provisions 2 cluster nodes and making a remote interaction. 
1. seed is a member node and provision no services of its own. 
2. then microservices variable is a member that joins seed member and provision GreetingService instance.
3. finally from seed node - create a proxy by the GreetingService api and send a greeting request. 

```java
    //1. ScaleCube Node node with no members
    Microservices seed = Microservices.builder().startAwait();

    //2. Construct a ScaleCube node which joins the cluster hosting the Greeting Service
    Microservices microservices =
        Microservices.builder()
            .discovery(
                self ->
                    new ScalecubeServiceDiscovery(self)
                        .options(opts -> opts.seedMembers(toAddress(seed.discovery().address()))))
            .transport(ServiceTransports::rsocketServiceTransport)
            .services(new GreetingServiceImpl())
            .startAwait();

    //3. Create service proxy
    GreetingsService service = seed.call().api(GreetingsService.class);

    // Execute the services and subscribe to service events
    service.sayHello("joe").subscribe(consumer -> {
      System.out.println(consumer.message());
    });
```

Basic Service Example:


* RequestOne: Send single request and expect single reply
* RequestStream: Send single request and expect stream of responses.
* RequestBidirectional: send stream of requests and expect stream of responses.

A service is nothing but an interface declaring what methods we wish to provision at our cluster.

```java

@Service
public interface ExampleService {

  @ServiceMethod
  Mono<String> sayHello(String request);

  @ServiceMethod
  Flux<MyResponse> helloStream();

  @ServiceMethod
  Flux<MyResponse> helloBidirectional(Flux<MyRequest> requests);
}

```

## API-Gateway:

Available api-gateways are [rsocket](/services-gateway-rsocket), [http](/services-gateway-http) and [websocket](/services-gateway-websocket)

Basic API-Gateway example:

```java

    Microservices.builder()
        .discovery(options -> options.seeds(seed.discovery().address()))
        .services(...) // OPTIONAL: services (if any) as part of this node.

        // configure list of gateways plugins exposing the apis
        .gateway(options -> new WebsocketGateway(options.id("ws").port(8080)))
        .gateway(options -> new HttpGateway(options.id("http").port(7070)))
        .gateway(options -> new RSocketGateway(options.id("rsws").port(9090)))
        
        .startAwait();
        
        // HINT: you can try connect using the api sandbox to these ports to try the api.
        // http://scalecube.io/api-sandbox/app/index.html
```

### Maven

With scalecube-services you may plug-and-play alternative providers for Transport,Codecs and discovery. 
Scalecube is using ServiceLoader to load providers from class path, 
  
You can think about scalecube as slf4j for microservices - Currently supported SPIs: 

**Transport providers:**

* scalecube-services-transport-rsocket: using rsocket to communicate with remote services.

**Message codec providers:**

* scalecube-services-transport-jackson: using Jackson to encode / decode service messages. https://github.com/FasterXML
* scalecube-services-transport-protostuff: using protostuff to encode / decode service messages. https://github.com/protostuff
 
**Service discovery providers:**

* scalecube-services-discovery: using scalecue-cluster do locate service Endpoint within the cluster
   https://github.com/scalecube/scalecube-cluster
    

Binaries and dependency information for Maven can be found at http://search.maven.org.

https://mvnrepository.com/artifact/io.scalecube

To add a dependency on ScaleCube Services using Maven, use the following:

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.scalecube/scalecube-services-api/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.scalecube/scalecube-services-api)

```xml

 <properties>
   <scalecube.version>2.x.x</scalecube.version>
 </properties>

 <!-- -------------------------------------------
   scalecube core and api:   
 ------------------------------------------- -->

 <!-- scalecube apis   -->
 <dependency>
  <groupId>io.scalecube</groupId>
  <artifactId>scalecube-services-api</artifactId>
  <version>${scalecube.version}</version>
 </dependency>
 
 <!-- scalecube services module   -->
 <dependency>
  <groupId>io.scalecube</groupId>
  <artifactId>scalecube-services</artifactId>
  <version>${scalecube.version}</version>
 </dependency>
 

 <!--

     Plugins / SPIs: bellow a list of providers you may choose from. to constract your own configuration:
     you are welcome to build/contribute your own plugins please consider the existing ones as example.

  -->

 <!-- scalecube transport providers:  -->
 <dependency>
  <groupId>io.scalecube</groupId>
  <artifactId>scalecube-services-transport-rsocket</artifactId>
  <version>${scalecube.version}</version>
 </dependency> 
```

## Overview 
Create distributed applications by breaking your application apart into decoupled services that can be built and scaled separately.

ScaleCube Microservices is a message-driven and asynchronous by default, built to scale due to its peer-to-peer nature. 
Powered by scalecube-cluster gossip capabilities to answer the cross-cutting-concerns such as: canary testing, service discovery, location transparency, 
fault-tolerance and real time failure-detection. 
ScaleCube Microservices provides fluent java functional APIs and Reactor Project Mono benefits. 
The ScaleCube Microservices are lightweight and embeddable in-order to reduce restrictions regards service implementation; 
it only requires a simple declaration of the Service APIs as an entry point to the service component. 
With ScaleCube Microservices its possible to provision services within same process, mulitple processes on the same hardware or mulitple processes on mulitple hardwares thus enabling ease of development and testing of a distributed system.

### Basic Concepts:

### Microservice
A Lightweight system component running on it's own process. 
Built around business capabilities and independently deployable by fully automated deployment machinery. 
Isolated, distributed and have well bounded context, communicates via APIs. Member in a microservices cluster, discoverable and faliure aware.

### Service Discovery
With ScaleCube Microservices each Microservice self-registeres at the cluster and utilizes the cluster gossip-protocol to declare and register so it can transparently communicate with its peer microservices that share the same cluster group.

### Routing
Microservice may have more then one instance or version of an instance at the cluster, the routing strategy is resposible to select the relevant microservice instance when invoking a service. It can be but not limited to: Round-robin, random selection, session based selection etc.

### Service Proxy
Microservices are message driven and communicate via well defined service interface APIs. The consumer uses a service proxy as a client to the microservices generated from the Microservice interface.


# Defining A Services
ScaleCube services are described by an interface. This interface defines how the service is invoked and implemented. Generally, the service interface, its implementation and consumption should remain agnostic to what transport is being used, it mainly defines the service API and data-model. the service interface is simple java interface annotated with a @Service and @ServiceMethod ScaleCube annotaions. The service interface is a simple java interface with one or more service methods. A service method nust return Mono or Flux or void in case no result is expected.

since microservices are message driven (as opposed to RPC) the service method can accept only one paramter that might be either Pojo data transfer object (DTO) or flux meaning an inbound stream of messages (DTOs)

This allows developers to compose the following service interaction models:
- Request-Response
- Fire And Forget
- Request Stream
- Bidirectional Request Stream.

```java
  @Service
  public interface GreetingService {
    
    // FIRE AND FORGET
    @ServiceMethod
    void greeting(GreetingRequest request);
    
    // REQUEST RESPONSE
    @ServiceMethod
    Mono<GreetingResponse> greeting(GreetingRequest request);

    // REQUEST STREAM
    @ServiceMethod
    Flux<GreetingResponse> greetingStream(GreetingRequest request);

    // REQUEST BIDI STREAM
    @ServiceMethod
    Flux<GreetingResponse> greetingChannel(Flux<GreetingRequest> requests);
    
  }
 ```

Service requests might be a local or a remote call and should not block the service consumer (unless the consumer explicitly called the Mono.block() method).
 

## Service Annotations

#### @Service annotation 
is used to define the service name. By default if a user defined name is not specified, 
the class name is used as the service logical name. Optionally, a user may specify a logical name of a service: 

```java
@Service ("my-specific-service-name")
```

The logical service name is used to address the service in the cluster when it is registered and discovered. 
Each service should have its own unique logical name in a given cluster / tenant.

#### @ServiceMethod annotation 
is used to define a service method. By default if a logical name is not specified, the method name is used as the service method logical name. Optionally, a user may specify a logical name of a service method:

```java
@ServiceMethod ("specific-method-name")
```

Each service method should have its own unique logical name for a given service.

















----

## Sponsored by [OM2](https://www.om2.com/)
