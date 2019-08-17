# Microservices

### Table of Contents

  * [Configuration Server](#Configuration-Server) - [Tutorial](#config-server-tutorial)
  * [Service Registry](#Service-Registry) - [Tutorial](#service-registry-tutorial)
  * [Queues](#Queues) - [Tutorial](#tutorial-spring-rabbitmq-tutorial)
  * [Cache](#cache) - [Tutorial](#tutorial-spring-data-caching-tutorial)
  * [Edge Service](#edge-service) - [Tutorial](#tutorial-edge-service--feign-tutorial)
  * [Circuit Breaker](#circuit-breaker-pattern) - [Tutorial](#tutorial-1)

## Configuration Server

**What are the advantages of externalizing configuration settings?**

* Settings can be changed without having to rebuild code.
* Settings can be changed without having to restart the application.
* Settings are centralized.
* We can trace changes to settings.
* We can provide encryption and decryption of sensitive settings.


### [Config Server Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/config-server-tutorial.md)

### Tutorial Summary:

#### How to setup configuration server?

1. Create a new (or use an existing) GitHub repository to store application configuration files.
1. Create a new IntelliJ project called `config-server` for the configuration server using Spring Initializr with the `Config Server` dependency.
1. Add the `@EnableConfigServer` annotation to the main project class.
1. Define the `server.port` and `spring.cloud.config.server.git.uri` for Github in the `application.properties` file.

		server.port=9999
		spring.cloud.config.server.git.uri=https://github.com/Ahmed3lmallah/configServer.git
		spring.cloud.config.server.git.username: username
		spring.cloud.config.server.git.password: password

1. Start the server and visit ```http://localhost:9999/properties-file-name/master```.

#### How to utilize a configuration server?

1. Add the `Config client` and `Spring Boot Actuator` dependencies to the service.
		
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		
1. Add `app-name-service.properties` file to the GitHub repository.
	1. Include server port, refresh scope configuration, and any other properties we would need to configure the service

			
			server.port=8080
			# Allow for RefreshScope
			management.endpoints.web.exposure.include=*
 
1. Add `bootstrap.properties` file to the resources folder
	1. include the config server uri
	1. include the service name (**Must match the properties file name in Github**)
	
			spring.application.name=service-name-service
			spring.cloud.config.uri=http://localhost:9999

1. Add `@RefreshScope` to the controller class
1. To extract predefined variables from the properties file we can use: 
		
		@Value("${officialGreeting}")
		private String officialGreeting;

1. Start the service.

*To update the service Beans if changes were made to the .properties file we can make use of `@RefreshScope` by using `$ curl localhost:8080/actuator/refresh -d {} -H "Content-Type: application/json"`*

## Service Registry 

### [Service Registry Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/service-registry-tutorial.md)

This tutorial steps you through setting up a Eureka Service Registry and the creation of a system that uses the Service Registry to locate a remote service and then call it.

### System Design

![image-20190620233725325](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/images/hello-cloud-system-v2.png)

* Service 1 is a REST web service that uses Config Server for its configuration files. 
* Service 1 registers with the Service Registry when it starts up.
* Service 2 will utilize Service 1.
* Service 2 asks the Service Registry for the location and connection details for Service 1 and uses that information to call it's controller method.

### How to setup Service Registry?

1. Create a new IntelliJ project for the service registery using Spring Initializr (start.spring.io)
	1. Name it `eureka-service-registry`
	1. Use `Eureka Server` dependency
1. Add `@EnableEurekaServer` annotation to the main application class

		@SpringBootApplication
		@EnableEurekaServer
		public class EurekaServiceRegistryApplication {

			public static void main(String[] args) {
				SpringApplication.run(EurekaServiceRegistryApplication.class, args);
			}
		}

1. Set the host name and port on which our Eureka Server will listen and configure some Eureka specific settings in the `application.properties` file.

		server.port=8761
		eureka.instance.hostname=localhost

		# Shut off the client functionality of the Eureka server (used for HA)
		eureka.client.registerWithEureka=false
		eureka.client.fetchRegistry=false
		eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka

	The value for ```server.port``` is arbitrary. For our demonstration, the hostname is ```localhost``` because we are running it locally. The ```registerWithEureka``` and ```fetchRegistry``` setting turn off the high availability/clustering features of Eureka because we are only running a single node.

1. Start your new Eureka Service Registry and visit ```http://localhost:8761```. You should see a Spring Eureka System Status page is everything is running correctly.

### How to use Service Registry?

1. Create a new IntelliJ project for the service using Spring Initializr (start.spring.io)
	1. Name it `service-name-service`
	1. Use `Config Client`, `Spring Web Starter`, `Eureka Discovery Client`, and `Spring Boot Actuator` dependencies
1. Create the Service Configuration File ```service-name-service.properties``` and add it to the GitHub repository

	**The Config Server matches properties files to their respective services by name: the name of the configuration file must match the application name**

		# this is the port on which our random-greeting-service will run
		server.port=2121

		# allow for RefreshScope
		management.endpoints.web.exposure.include=*
		
	**As indicated by the comments, our service (random-greeting-service) will run on port 2121 (this value is arbitrary). The next entry is needed to enable us to force our service to re-read its configuration without having to restart**

1. Configure the Service to Use the Config Server
	1. Create a file called ```src/main/resources/bootstrap.properties``` in the project and add the following entries:

			# This file has just enough information so that our application can find the configuration
			# service and its configuration settings.

			# This name must match the name of the properties file for this application
			# in the configuration repository. we are looking for a file called hello-cloud-config.properties
			spring.application.name=service1-name-service

			# This is the url to the configuration service that we will use to get our configuration
			spring.cloud.config.uri=http://localhost:9999

	**The ```spring.application.name``` property value must match the name of the properties file we checked into Git.**
	**The host and port we use for the ```spring.cloud.config.uri``` must match the host and port values on which our Config Server is running**
	
1. Create the Controller

		@RestController
		@RefreshScope
		public class RandomGreetingServiceController {

			.
			.
			.
		}

1. Configure Random Greeting Service to Register with Eureka
	1. Add the  ```@EnbleDiscoveryClient``` class level annotation to the main application class.
			
			@SpringBootApplication
			@EnableDiscoveryClient
			public class RandomGreetingServiceApplication {

				public static void main(String[] args) {
					SpringApplication.run(RandomGreetingServiceApplication.class, args);
				}
			}

### Creating Service 2 that utitlizes Service 1 registered with Service Registry

1. Add `Eureka Client` Dependency or manually add the following to an existing project POM.xml file

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

1. Add ```@EnableDiscoveryClient``` class level annotation to the main application class.

	**Add this annotation does the following:**

	1. Causes the Hello Cloud Service to register with the Eureka Service registry upon startup.
	1. Allows the Hello Cloud Service to contact the Eureka Service Registry to lookup the Random Greeting Service location and connection information.

1. Add Configuration Entries to the ```service2-name-service.properties``` file. 

		server.port=7979

		# allow for RefreshScope
		management.endpoints.web.exposure.include=*

		officialGreeting="Greetings from the Hello Cloud Service!!! We're glad you're here!"

		randomGreetingServiceName=service1-name-service
		serviceProtocol=http://
		servicePath=/greeting

	The new entries will help us locate and call Service 1.

1. Create Controller

		@RestController
		@RefreshScope
		public class HelloCloudServiceController {

			@Autowired
			private DiscoveryClient discoveryClient;

			private RestTemplate restTemplate = new RestTemplate();

			@Value("${randomGreetingServiceName}")
			private String randomGreetingServiceName;

			@Value("${serviceProtocol}")
			private String serviceProtocol;

			@Value("${servicePath}")
			private String servicePath;

			@Value("${officialGreeting}")
			private String officialGreeting;

			@RequestMapping(value="/hello", method = RequestMethod.GET)
			public String helloCloud() {

				List<ServiceInstance> instances = discoveryClient.getInstances(randomGreetingServiceName);

				String randomGreetingServiceUri = serviceProtocol + instances.get(0).getHost() + ":" + instances.get(0).getPort() + servicePath;

				String greeting = restTemplate.getForObject(randomGreetingServiceUri, String.class);

				return greeting;
			}
		}


	Things to note about this code:

	1. We autowire a DiscoveryClient instance. We'll use this to contact Eureka and ask about the connection details of Service 1.
	2. We include a RestTemplate property. This will used to communicate with the Service 1. It allows us to make REST calls from our Java code.
	3. We use the ```@Value``` annotation to get the values of the Service 1, serviceProtocol, and servicePath properties in our config file.
	4. ```helloCloud``` Method:
	   1. We use the DiscoveryClient to ask for Service 1 by name.
	   2. We combine the serviceProtocol and servicePath from our configuration file with the host and port of Service 1 from Eureka to create the URI for the Service 1.
	   3. We use the restTemplate and the URI to call the Service 1 and get our greeting.

### Running the Service Registry System

Start the services in the following order:

1. config-service
2. eureka-service-registry
3. service 1
4. service 2 (utilizing service 1)

## Queues 

**Asynchronous vs. synchronous processing:** 

Synchronous processing means that you can only execute one process at a time, while Asynchronous processing means multiple process can be executed at a time and you don't have to finish executing the current process in order to move on to next one.

**What is a Queue?**

**Queue** is a **linear data structure, or an ordered list of elements of similar data types**. in which the first element is inserted from one end called the **REAR (also called tail)**, and the removal of existing element takes place from the other end called as **FRONT (also called head)**. This makes queue as **FIFO (First in First Out) data structure**, which means that element inserted first will be removed first. The process to add an element into queue is called Enqueue and the process of removal of an element from queue is called Dequeue.

**What problems do Queues solve?**

1. Queues enable asynchronous communication, which means that the endpoints that are producing and consuming messages interact with the queue, not each other. Producers can add requests to the queue without waiting for them to be processed. Consumers process messages only when they are available. No component in the system is ever stalled waiting for another, optimizing data flow.

1. Queues make your service reliable, and reduce the errors that happen when different parts of your system go offline. By separating different components with message queues, you create more fault tolerance. If one part of the system is ever unreachable, the other can still continue to interact with the queue. The queue itself can also be mirrored for even more availability.

**How Does a Queue Work?**

1. Producer creates a new entry/message.
	* Entry/message includes the exchange name and a binding key (routing key).
1. Entry/message is sent to a message broker such as RabbitMQ.
1. Entry/message is routed to the appropriate queue(s) based on the binding key and distribution protocols.
1. Consumer listening to the Queue receives the message when available.

### Terminology

It's helpful to review queue-related terminlogy before we look at the design and begin implementation.

#### AMQP

Advanced Message Queuing Protocol (**AMQP**) is a messaging protocol that allows clients and messaging middleware to communicate in a standardized manner. RabbitMQ and the Spring client libraries conform to AMQP.

#### Producer

The **producer** is the application, process, or code that creates new entries and places them in the queue for processing.

#### Consumer

The **consumer** is the application, process, or code that processes, or consumes, the queue entries created by the producer.

#### Queue

A **queue** is a repository for messages. Producers place the messages in the queue. Consumers process the messages.

#### Exchange

An **exchange** is where producers send messages in an AMQP system. An exchange then routes the messages to one or more queues based on the type of exchange and routing rules (called bindings, explained below).

**Topic exchanges** route messages to one or more queues based on a routing key and the pattern used to bind the exchange to the queue or queues. Topic exchanges allow consumers to choose which types of messages they want to process and which ones they want to ignore.

#### Binding

A **binding** is a rule that an exchange uses to route messages to a queue. Routing keys and binding rules can be used to filter messages and only send certain messages to certain queues.

**Additional Resources:**

* [Message Queues - AWS](https://aws.amazon.com/message-queue/)

### Tutorial: [Spring RabbitMQ Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-rabbitmq-tutorial.md)

### Tutorial Summary:

1.
1.
1.

* To use RabbitMQ GUI: 

1. Go to root directory. Default: C:\Program Files\RabbitMQ Server\rabbitmq_server-3.7.16\sbin>
1. Use RabbitMQ CMD
1. `rabbitmq-plugins enable rabbitmq_management`
1. Restart the rappitmq server
1. visit localhost:15672/
1. Login with username: `guest` and Password: `guest`

## Cache

**What Is a Cache?**

A **cache is a copy of the original data** that is stored for future reference. Previously accessed data is stored in a cache to make future retrieval faster.

**Why Use a Cache?**

* It’s less expensive.
	* Saves money
	* Saves processing time
* Data transformation requires processing power. Once that data is transformed, caching the results saves our app from have to process it again.
* Improved user experience.

**How Does a Cache Work?**

1. The app attempts to fetch the data from the cache.
1. If the data exists in the cache, it is presented.
1. If the data is not in the cache, the data is requested from the database.
1. This data may then be cached for faster retrieval when future requests are made.

**Things to Consider When Using a Cache**

* Caches take up disk space.
* Caches are most useful with generic data; i.e., not user-specific.
* Cached data may not be the most recent, therefore not accurate.
	* If the accuracy of the data is critical, a cache is likely not the best solution.
* When data changes often, a cache is likely not very effective.

**Difference between Cache and buffer:**

**Caching** is accessing a copy of the data, while **Buffering** preloads data from the “original” source. Buffering is usually used with large file sizes to match the transmission speed of the sender and the receiver.

**Additional Resources:**

* 
* 
* 

### Tutorial: [Spring Data Caching Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-caching-tutorial.md)

### Tutorial Summary:

1. 
1.
1. 

## Edge Service

**What Is a Edge Service?**

**Edge Service** is an **entry point into a service that sits in front of an API.** In other words, it acts as a gateway to other services. Edge service is usually exposed to the public internet and is used to route HTTP requests to the appropriate service(s).

**Why Use an Edge Service?**

* Benefits
	* Allows us to request from multiple services with a single request
	* Encapsulates the internal structure of the app
	* May improve security through authentication
	* May improve performance through load balancing
	* Improves the user experience
	* Can be tailored to specific clients

* Concerns
	* Another component to manage and update

**How Does an Edge Service Work?**

1. A client makes a request
1. Looks for a microservice (or multiple microservices) to deliver the request
1. Multiple microservices are invoked and aggregated
1. Returns the response to the caller

**Additional Resources:**

* 
* 
* 

### Tutorial: [Edge Service / Feign Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/edge-service-feign-tutorial.md)

### Tutorial Summary:

1. 
1.
1.

## Circuit Breaker Pattern

**Why Use the Circuit Breaker Pattern?**

* Remote calls can fail or “hang” until a timeout is reached. 
* Running out of resources can lead to cascading failures.

**How Circuit Breakers Work?**

You wrap a protected function call in a circuit breaker object, which monitors for failures. Once the failures reach a certain threshold, the circuit breaker trips, and all further calls to the circuit breaker return with an error, without the protected call being made at all. Usually you'll also want some kind of monitor alert if the circuit breaker trips.

![image](https://martinfowler.com/bliki/images/circuitBreaker/sketch.png)

Circuit breaker status should be monitored and logged, and we must consider what to do when the breaker fails!

For software circuit breakers, we can have the breaker itself detect if the underlying calls are working again. We can implement this self-resetting behavior by trying the protected call again after a suitable interval, and resetting the breaker should it succeed.

![image](https://martinfowler.com/bliki/images/circuitBreaker/state.png)

Circuit breakers can also be used for asynchronous communications:
* Put all requests on a queue
* Open the circuit when the queue is full

**Netflix Hystrix:**

“Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.”

**Additional Resources:**

* [Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
* [Fault Tolerance in a High Volume, Distributed System](https://medium.com/netflix-techblog/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a)
* [Netflix/Hystrix Git Repo](https://github.com/Netflix/Hystrix)

### [Tutorial 1](https://spring.io/guides/gs/circuit-breaker/)

### Tutorial 1 Summary:

1. 
1.
1.

### [Tutorial 2](https://www.baeldung.com/spring-cloud-netflix-hystrix)

### Tutorial 2 Summary:

1. 
1.
1.


#### [Go Back...](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)