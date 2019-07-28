# Microservices

### Table of Contents

  * [Configuration Server](#Configuration-Server-tutorial)
  * [Service Registry](#Service-Registry-tutorial)
  * [Queues Tutorial](#Queues-Spring-RabbitMQ-Tutorial)
  * [Cache Tutorial](#cache-spring-data-caching-tutorial)
  * [Edge Service Tutorial](#edge-service-feign-tutorial)

## Configuration Server [Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/config-server-tutorial.md)

### How to setup configuration server?

1. Create a new public (read only) GitHub repository to store application configuration files.
1. Create a new IntelliJ project for the configuration server using Spring Initializr (start.spring.io)
	1. Name it `app-config-server`
	1. Use `Config Server` dependency
1. Add the `@EnableConfigServer` annotation to the main project class.
		
		@SpringBootApplication
		@EnableConfigServer
		public class AhmedElMallahCloudConfigServerApplication {

			public static void main(String[] args) {
				SpringApplication.run(AhmedElMallahCloudConfigServerApplication.class, args);
			}
		}

1. Define the config-server port and url for Github in the `application.properties` file.

		server.port=9999
		spring.cloud.config.server.git.uri=https://github.com/Ahmed3lmallah/configServer.git

1. start the server and visit ```http://localhost:9999/properties-file-name/master```.

### How to use configuration server?

To create a client microservice that utlilizes the client server we need to follow the following steps:

1. Create a new IntelliJ project for the configuration server using Spring Initializr (start.spring.io)
	1. Name it `service-name-service`
	1. Use `Config client`, `Spring Boot Actuator`, and `Spring Web Starter` dependencies
1. Add the `@EnableDiscoveryClient` annotation to the main project class.
		
		@SpringBootApplication
		@EnableDiscoveryClient
		public class AhmedElMallahMagicEightBallServiceApplication {

			public static void main(String[] args) {
				SpringApplication.run(AhmedElMallahMagicEightBallServiceApplication.class, args);
			}
		}
1. Add `service-name-sevice.properties` file to the GitHub repository.
	1. Include server port, refresh scope configuration, and any other properties we would need to configure the service

			# this is the port on which our random-greeting-service will run
			server.port=3344

			# allow for RefreshScope
			management.endpoints.web.exposure.include=*
 
1. Add `bootstrap.properties` file to the resources folder
	1. include the config server uri
	1. include the service name (**Must match the properties file name in Github**)
	
			# This file has just enough information so that our application can find the configuration
			# service and its configuration settings.

			# This name must match the name of the properties file for this application
			# in the configuration repository. we are looking for a file called hello-cloud-config.properties
			spring.application.name=service-name-service
			# This is the url to the configuration service that we will use to get our configuration
			spring.cloud.config.uri=http://localhost:9999

1. Add `@RefreshScope` to the controller class using the service
1. To extract predefined variables from the properties file we can use: 
		
		@Value("${officialGreeting}")
		private String officialGreeting;

1. Start the server and visit ```http://localhost:3344```.

## Service Registry [Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/service-registry-tutorial.md)

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

## Queues [Spring RabbitMQ Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-rabbitmq-tutorial.md)

## Cache [Spring Data Caching Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/spring-caching-tutorial.md)

## Edge Service [Feign Tutorial](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/Tutorials/edge-service-feign-tutorial.md)

#### [Go Back](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)