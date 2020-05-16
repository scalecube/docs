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
