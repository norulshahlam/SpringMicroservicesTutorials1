# 2021 edition - Java, Spring Framework, Spring Boot, Spring Cloud, Microservices, IntelliJ, REST, JPA, Maven, Zuul,Ribbon
# Part 1

GIT:

[https://github.com/futurexskill/java-springboot-microservices]


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
    CourseClient - http://localhost:8001/
                    http://localhost:8001/courses
                    http://localhost:8001/1
                    http://localhost:8001/2

    
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

`3. run this (in order)`

    
    EurekaServer - service discovery - http://localhost:8761/
      CourseClient - http://localhost:8001/
                    http://localhost:8001/courses
                    http://localhost:8001/1
                    http://localhost:8001/2
    CourseCatalog - http://localhost:8002/
                    http://localhost:8002/catalog
                    http://localhost:8002/firstcourse


## Implementing Circuit Breaker with Netflix Hystrix

Netflix Hystrix is a framework based on Circuit Breaker Design Pattern that help to make application fault tolerant and improves resilience. This is achieved by isolating failure and preventing them from impacting other features of the application i.e. fail fast and recover asap.

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

     CourseCatalog - http://localhost:8002/

then stop CourseClient and use CourseCatalogue service to fetch CourseClient again. Both will give different results, so now if CourseClient is down, it is being handled


## Building the User App

This will be registered in Eureka server/

`url lesson:`
  [https://www.udemy.com/course/java-spring-boot-microservices-project-for-beginners/learn/lecture/18103845#overview]

`project name:`
    UserService

`3. run this (in order)`
    
    EurekaServer - service discovery - http://localhost:8761/
    CourseClient - http://localhost:8001/
                    http://localhost:8001/courses
                    http://localhost:8001/1
                    http://localhost:8001/2
    CourseCatalog - http://localhost:8002/
    UserService - http://localhost:8003/
                http://localhost:8003/1
                http://localhost:8003/2


## Enhancing the Course Catalog App to fetch data from the User App

CourseCatalog will fetch course info from CourseClient and corresponding user info from user-service.


`1. run this (in order)`
    
    EurekaServer - service discovery - http://localhost:8761/
    CourseClient - http://localhost:8001/
                    http://localhost:8001/courses
                    http://localhost:8001/1
                    http://localhost:8001/2
    CourseCatalog - http://localhost:8002/
    UserService - http://localhost:8003/
                http://localhost:8003/1
                http://localhost:8003/2
    CourseCatalog - http://localhost:8002/catalog
                    http://localhost:8002/firstcourse


********************     end    ************************