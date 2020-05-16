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

---

## Implementing a services
Implementing a service is simple as implementing a java service interface. using pure java, so your model and implementation stays clean. One more benefit of such an approach comes when testing your service. Building the service is just like building any other java component. The service is a simple java class with one or more service methods. a service method may return Reactor Project Mono | Flux or void in case no result is expected. Service requests might be a local or remote call and should not block the service consumer (unless the consumer explicitly called the Mono.block())

NOTE: when implementing a reactive service, blocking a service call is not recommended. 

```java
import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

public class SimpleGreetingService implements GreetingService {
    
    @ServiceMethod   // FIRE AND FORGET
    void greeting(GreetingRequest request){
       // do somthing about it
    }
    
    
    @ServiceMethod   // REQUEST RESPONSE
    Mono<GreetingResponse> greeting(GreetingRequest request) {
       return Mono.just(new GreetingResponse())
    }

   
    @ServiceMethod   // REQUEST STREAM
    Flux<GreetingResponse> greetingStream(GreetingRequest request) {
       return Flux.empty()
    }

    
    @ServiceMethod   // REQUEST BIDI STREAM
    Flux<GreetingResponse> greetingChannel(Flux<GreetingRequest> requests){
        // do somthing about it
    }
}
```

---

## Provisioning Services
So far we have learned how to define and implement a service, actually it was nothing more than implementing a Java component.
In this section we will learn how to provision our components as clustered Microservices .

```java
  // Create microservice provider
  Microservices provider = Microservices.builder()
    .discovery(options -> options.seeds(....))
    .services(new GreetingServiceImpl(),...)
    .startAwait();
```

The line above introduces the service component to the cluster, it reads the information from the service interface and registers the instance in the cluster using the .services(...) option. It iss possible to introduce many service instances to the cluster or clusters for example by running several JVM instances, each containing a service instance or having many services in the same JVM instance.

### Service Tags
It is also possible to register services with service tags. The service .tag(key,value) is a user defined property used to describe a service instance. Tag helps to distinguish instances and a single instance, it is possible to add several tags to the service description.
Following are several examples use-cases of Service Tags:
- Route and select a specific service instance 30% of the times.
- Assign a specific service instance with special role in the cluster.
- Always choose the latest version of a service.
- ...

```java
    Microservices services1 = Microservices.builder()
        .discovery(options -> options.seeds(....))
        .service(ServiceInfo.fromServiceInstance(new GreetingServiceImpl())
              .tag("Weight", "0.3")
              .tag("Version", "1.0.3")
              .tag("Role", "Master")
            .build())
        .startAwait();
```
---

## Consuming Services
We have seen how to define a service interfaces and how to implement them, now we need to consume them. The service interface contains everything ScaleCube services needs to know on how to go about invoking a service.

A service implementation may be located anywhere at the cluster and or may appear with number of instances. ScaleCube service ServiceCall solve this for you. no matter where and how much service instances you have it gives you the full control to consume them.

A service consumer is a member of the service cluster sharing same Cluster Membership / Failure Detection / Gossip discovery group. This can be achieved by joining one of the known cluster members AKA .seeds() nodes.

```java
// Create microservice consumer
Microservices consumer = Microservices.builder()
		.discovery(options -> options.seeds(....))
		.startAwait();
```

Using the service interface class we can request from the consumer a ServiceCall for a given .api(GreetingService.class). The call to the api method will build for us a service proxy with relevant descriptors to address the specific service in the cluster.

```java
  // Get a proxy to the service API.
  ServiceCall serviceCall = consumer.call().create();
  
  // Creates a service proxy passing the service interface class
  GreetingService greetingService = serviceCall.api(GreetingService.class);
```

Using the service proxy to call the concrete service is as trivial as a simple method call. The service proxy will locate the service   instance in the cluster and will route to them the requests. Services supports java Reactor Project Mono for a async request response   and Flux for a reactive streams pattern.

```java
  // Call service and when complete print the greeting.
  GreetingRequest req = new GreetingRequest("Joe");
  
  Publisher<GreetingResponse> publisher = greetingService.greeting(req);
  GreetingResponse response = 
		Mono.from(publisher).subscribe(result -> {
			System.out.println(result.greeting());
		});
  
  Flux<GreetingResponse> stream = greetingService.greetings(req)
   .subscribe(onNext -> {
      System.out.println(onNext.getResult())
  });
```

By default the scale-cube services provides a Round-robin service instance selection. Thus for each service request the service proxy will use the currently available service endpoints and will invoke one message to one service instance at a time so its balanced across all currently-live-service-instances. Using the .router(...) its also possible to control the endpoint selection logic.

Service Proxy Options:
.router(...) option you can choose from available selection logic or provide a custom one.
.timeout(Duration) option you can specify a timeout for waiting for service response the default is 30 secounds.

```java
  // Get a proxy to a service API via a router.
    CanaryService service = gateway.call()
        .router(Routers.getRouter(CanaryTestingRouter.class))
        .create()
        .api(CanaryService.class);
```




----

## Sponsored by [OM2](https://www.om2.com/)
