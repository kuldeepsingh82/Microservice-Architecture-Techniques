
# Microservice-Architecture-Techniques

Consider reading the mentioned recommendation before starting any microservices bases architecture

  

## What are MicroServices

Fine grained loosely coupled services
  

  

## Advantages of MicroServices

* Loose Coupling

* Improve Modularity

* Scalable

* Parallel Development

  

## Disadvantages of MicroServices

* Infrastructure cost is higher

* More development efforts

* Service management is difficult

## Do / Don't
* Plan your project properly
* Decide how to decompose the micro-services
* Ensure you have a micro-service project template with all common functionality included
* Setup a CICD pipeline in the beginning of the project
* Decide the communication protocol between the microservices 
* Setup test automation for unit testing / integration testing of microservices
 
* Avoid Nano Anti Pattern (Too fine grained micro services)

## Microservice Template

A startup project containing cross-cutting concerns

* Include logging services (with common ticketId)
* Include database configurations (if applicable)
* Include circuit breaker configuration (such as [Hystrix](https://github.com/Netflix/Hystrix))
* Setup security for microservices

## Decompose Microservices & Source Code Repositories

* Divide your project into multiple sub-projects. A sub-project can hold multiple microservices.
* Avoid nano microservices while decomposing the services
* Keep related/dependent microservices together to avoid deployment mistakes 
* Use business capabilities to decompose the microservices (For instance for ecommerce based projects, cart service, product service, payment service etc)
* Create a repository for each microservice inside a sub-project

## Inter-Service Communication

* Rest APIs
* RPC
* Message Broker based communication (such as Kafka or ActiveMQ)

**Recommended Solution** : Mixup of both. Rest APIs and RPC are synchronous in nature, so where sync communication is required for either RestAPIs or RPC. Message/Event based communication is Async, use it where calling service doesn't bother about the response. 

## Service Registry & Discovery

To support scaling of the microservices it is required to setup a Service Registry & Service Discovery

### Service Registry :
Is a service which keeps records of all available services on the server. All microservices should inform the services registry whenever they are coming up or going down. Additionally, all microservices should expose a health api which services registry can use to check the availability of the micro services

### Service Discovery :
Is a service which provide connection information about the required service. This can be used by client-side discovery to find the address of the microservice (Same as DNS Server) OR it can be used by server-side (such as load balancer) to redirect the incoming request to appropriate microservice.

**Recommended Solution** : Kubernetes with docker build which not only provide Ingress service (with load balancer) but also manage auto-scaling of the pods

## Database Decomposition 

Database need a special care in microservices based architecture 

* Shared Database (Not Recommended) : Where we have only one database and it is shared across microservices. 
* Database Per Service (Recommended) : Each microservice will have its own database.

### a. How to get shared/joined data?

Database per service is recommended for scaling but it poses a problem when it comes to get the data from multiple microservices. There are two good solutions to solve this 

* CompositeMicroService : Create another microservice which can fetch data from two different microservices then join the data in-memory and provide the required information. 
* Event Based Messaging (Event Sourcing) : Each microservice should produce a message (with data) which other other microservices can consume and prepare the joined data. Which then can be consumed by the microservices.

**Recommended Solution** : Mixup of both. For the microservices where updated data is required al the time & data is changing frequently, go for CompositeMicroService. Otherwise go for event-based messaging (such as Kafka or ActiveMQ)

### b. How to store Transactional data (Data Integrity)?

Database per service is recommended for scaling but it poses data integrity problem when it comes to store transactional data with the help of multiple microservices. There are two good solutions to solve this 

* **Two Phase Commit** : Create a co-ordinator microservice which can send the request to underlying microservices, they in-turn execute the command but do not commit the transaction. If all microservices returns ok, co-ordinator service can instruct them to commit the transaction. If all microservices commit is successful, co-ordinator service respond with success code else if any commit fails, co-ordinator service instruct all microservices to rollback the transaction.

* **Saga** : Each microservice produce a message which other other microservices can consume and store the data. In case any microservice fails it can send rollback event to all other microservices.

	- **Choreography based sagas** 
	
	- **Orchestrator based sagas** 

**Recommended Solution** : Mixup of both. For the microservices where updated data is required al the time & data is changing frequently, go for CompositeMicroService. Otherwise go for event-based messaging (such as Kafka or ActiveMQ)

## Failover Mechanism

Always consider adding following in the microservice project template

* **Circuit Breaker** : 
Fail fast technique to remove/disable the microservices from the system which are not available or not performing upto the mark. One can use [Netflix Hystrix](https://github.com/Netflix/Hystrix) for all microservices to ensure they respond in the specified time.


* **Logging across microservices** : 
All microservices will use their own logging files. Ensure a ticket Id is used and passed across microservices to track any given request. Additionally, use tools like [ELK](https://www.elastic.co/what-is/elk-stack) or [Slack](https://slack.com) to aggregate the logs in one place. Such tools also provide alerts and dashboard which are quite useful.


* **Health API** : 
Create a health check light weight API which can be used to check the health of the microservice by monitoring tools and service registry etc.
