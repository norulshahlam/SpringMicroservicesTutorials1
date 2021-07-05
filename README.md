# 2021 edition - Java, Spring Framework, Spring Boot, Spring Cloud, Microservices, IntelliJ, REST, JPA, Maven, Zuul,Ribbon

# Part 1

## overview

we have 3 service:

    EurekaServer - service discovery 
    CourseClient - registers on service discovery
    CourseCatalogue - access CourseClient thru service discovery

## Service discovery using Eureka

`url lesson:`
    [https://www.udemy.com/course/java-spring-boot-microservices-project-for-beginners/learn/lecture/18067189#overview]

`project name:`
    EurekaServer

Eureka Server is an application that holds the information about all client-service applications. Every Micro service will register into the Eureka server and Eureka server knows all the client applications running on each port and IP address. Eureka Server is also known as Discovery Server.

to `register a service as Eureka server`, simply:

    1. add pom:

        <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

    2. add @EnableEurekaServer in your main method to make it

    3. add in your application.properties:

        server.port=8761
        spring.application.name=fx-discovery-server
        eureka.client.register-with-eureka=false
        eureka.client.fetch-registry=false

    4. run to test:

        EurekaServer - service discovery - http://localhost:8761/

## Registering the Course app as a Eureka client

`url lesson:`
    [https://www.udemy.com/course/java-spring-boot-microservices-project-for-beginners/learn/lecture/18070999#overview]

`project name:`
    CourseClient

Now we will register this service as Eureka client. This service also uses sqlite for db. go to [https://sqlite.org/download.html] to get the tools to run sqlite. we have a tool here to run in windows env. so run CourseClient/sqlite-tools-win32-x86/sqlite3 in CMD!

`run sqlite`

    u can set path to run your sqlite anywhere.
    
       [https://www.udemy.com/course/java-spring-boot-microservices-project-for-beginners/learn/lecture/17987777#content]
    
    these are the queries:

    $sqlite3 futurexcourse.db

    .databases

    CREATE TABLE
        course
        (
            
            courseid BIGINT NOT NULL,
            coursename VARCHAR,
            author VARCHAR,
            PRIMARY KEY (courseid)
        );
        
        
    INSERT INTO course (courseid, coursename, author) VALUES 
    (1, 'Big Data Hadoop Spark', 'FutureXSkill');	

    INSERT INTO course (courseid, coursename, author) VALUES 
    (2, 'Java Microservices', 'FutureXSkill');	

    INSERT INTO course (courseid, coursename, author) VALUES 
    (3, 'AWS', 'XYZ');	

    select * from course;

`run and test service (in order)`

    EurekaServer - service discovery - http://localhost:8761/
    CourseClient - http://localhost:8761/

    
## Integrating the Course Catalog app with the Course app through the Eureka server

`url lesson:`
    [https://www.udemy.com/course/java-spring-boot-microservices-project-for-beginners/learn/lecture/18060615#overview]

`project name:`
    CourseCatalog


CourseCatalog will connect to CourseClient thru eureka. CourseCatalog does not register in eureka. how to do this?

`1. interact with eureka first - `

    create instance of eureka in your controller
            
        @Autowired
        private EurekaClient client;

`2. use this instance to find the services registered in eureka   `

    services are found by its app name defined in applicatin.properties. for this case, we are finding service named 'course-client'

        InstanceInfo instanceInfo = client.getNextServerFromEureka("course-client",false);


## Implementing Circuit Breaker with Netflix Hystrix

Netflix Hystrix is a framework based on Circuit Breaker Design Pattern that help to make application fault tolerant and improves resilience. This is achieved by isolating failure and preventing them from impacting other features of the application i.e. fail fast and recover asap.

![alt text](https://www.google.com/imgres?imgurl=https%3A%2F%2Fwww.nexsoftsys.com%2Farticles%2Fimages%2Fthe-circuit-breaker-pattern-implementation-using-netflix-hystrix.jpg&imgrefurl=https%3A%2F%2Fwww.nexsoftsys.com%2Farticles%2Fcircuit-breaker-pattern-implementation-using-netflix-hystrix.html&tbnid=q9xlvMSOQl5JsM&vet=12ahUKEwjviJyxiczxAhWX6XMBHfElA_IQMygAegUIARCuAQ..i&docid=notne4fnLK9TuM&w=1500&h=900&q=Circuit%20Breaker%20with%20Netflix%20Hystrix&ved=2ahUKEwjviJyxiczxAhWX6XMBHfElA_IQMygAegUIARCuAQ)

[https://www.google.com/imgres?imgurl=https%3A%2F%2Fwww.nexsoftsys.com%2Farticles%2Fimages%2Fthe-circuit-breaker-pattern-implementation-using-netflix-hystrix.jpg&imgrefurl=https%3A%2F%2Fwww.nexsoftsys.com%2Farticles%2Fcircuit-breaker-pattern-implementation-using-netflix-hystrix.html&tbnid=q9xlvMSOQl5JsM&vet=12ahUKEwjviJyxiczxAhWX6XMBHfElA_IQMygAegUIARCuAQ..i&docid=notne4fnLK9TuM&w=1500&h=900&q=Circuit%20Breaker%20with%20Netflix%20Hystrix&ved=2ahUKEwjviJyxiczxAhWX6XMBHfElA_IQMygAegUIARCuAQ]

Recap: 

    EurekaServer - service discovery 
    CourseClient - registers on service discovery
    CourseCatalogue - access CourseClient thru service discovery

now in case if CourseClient is down, we have to configure CourseCatalog to give 'fallback' response. let do that.

`1. add pom`

	<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
	</dependency>

`2. change spring boot version`

    The above dependencies requires Spring Boot >= 2.0.0.RELEASE and < 2.4.0.M1.

`3 add @EnableCircuitBreaker in main method`

`4 handle this in homepage controller`

    by adding   
    
        @HystrixCommand(fallbackMethod = "displayDefaultHome")

now try to use CourseCatalogue service to fetch CourseClient with all the instances running
then stop CourseClient and use CourseCatalogue service to fetch CourseClient again.

Both will give different results, but now if CourseClient is down, it is being handled




























***********************************************

`this is part 2 of microservice. we are learning about Externalized Configuration`

Spring Boot lets you externalize your configuration so that you can work with the same application code in different environments. You can use properties files, YAML files, environment variables, and command-line arguments to externalize configuration. Property values can be injected directly into your beans by using the @Value annotation, accessed through Spring’s Environment abstraction, or be bound to structured objects through @ConfigurationProperties.

`run in order`

EurekaServer - service discovery - http://localhost:8761/
Config server - Eureka client -
http://localhost:8888/order-service/default,
http://localhost:8888/payment-service/default

in config server, we have set the config url using git uri. this git have several service properties which we can access using the above url. now try to edit something and u can see the changes reflected on the browser

`create order and payment service`

Order service: http://localhost:8001/
Payment service: http://localhost:8002/

`Integrating Microservices with Spring Cloud Config Server`

[https://www.udemy.com/course/java-spring-boot-microservices-project-for-beginners/learn/lecture/18192604#content]

a. by simply adding @EnableDiscoveryClient in both services, we r now conecting to the config server. re run both services n run the above url to see the diff response

#### note

if u cant connect to config server, try to add this dependency:

    <dependency>
    <groupId>org.springframework.cloud</groupId> 	<artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>

b. also by adding application name, we can connect to the specific service in the git url:

spring.application.name=order-service
spring.application.name=payment-service

re run both services n run the above url to see the diff response

c. now try to change both services in git application.properties. run the config server to see the immediate changes:

http://localhost:8888/order-service/default
http://localhost:8888/payment-service/default

u will see the changes without restarting the server.but in each services, ie

Order service: http://localhost:8001/
Payment service: http://localhost:8002/

### Refreshing configuration without restarting microservices

u wont find the changes refelcted unless u restart both of this servers. to allow changs without resrtarting, simply do this:

add `@RefreshScope` in both controllers.
run this IN POSTMAN to refresh the changes:

POST METHOD http://localhost:8002/actuator/refresh
POST METHOD http://localhost:8001/actuator/refresh

it will return the properties that was changed

reload to see the changes reflected without restarting the server:

Order service: http://localhost:8001/
Payment service: http://localhost:8002/
