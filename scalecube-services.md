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
---- 

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
