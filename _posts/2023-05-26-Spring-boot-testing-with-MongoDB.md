---
title: "Spring boot testing with MongoDB using TestContainers"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - MongoDB
  - Spring Boot
  - TestContainers
  - NoSQL
---

When we need to test our spring boot application that is running MongoDB, it will by default connect to the MongoDB instance with the connection details in our `application.properties`. This might not be ideal, as every time we run the same tests, which might be inserting data, our collection grows if we don't clean it up regularly. Or our test might just fail, because we are inserting duplicating data. A cleaner approach to testing with MongoDB is to start an instance of it in a docker container every time we run the tests. This way, the MongoDB instance is also destroyed after our tests, so that we don't have to worry about accumulating data. It might seem a hassle to configure something like this, but actually junit-jupiter has an extension to do that automatically for us, called [TestContainers](https://www.testcontainers.org/).

To incorporate TestContainers to our spring boot application, we need to add the dependencies to our `build.gradle`.

```
testImplementation "org.testcontainers:testcontainers:1.18.1"
testImplementation "org.testcontainers:junit-jupiter:1.18.1"
testImplementation "org.testcontainers:mongodb:1.18.1"
```

I'm testing for MongoDB, so I have the `org.testcontainers:mongodb`, but if you are using other databases, you can find the list of supported containers and their dependency from under the `Modules` menu on [https://www.testcontainers.org/](https://www.testcontainers.org/).

And as mentioned above, we are running the database in containers, so naturally, we need to have docker installed and running. If you are not familiar with Docker, do checkout this article on [Containerizing with Docker explained](https://www.thecodinganalyst.com/knowledgebase/Containerizing-with-Docker-explained/).

For tests that you need to run testcontainers, simply annotate the class with `@TestContainers`. It will then scan all fields in the class that is annotated with the `@Container` and run the containers' lifecycle methods, i.e. to start them. 

The `@Container` will be the docker container of the MongoDB we want to start for running our tests. 

```
@Container
public static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo:latest").withExposedPorts(27017);
```

The `mongo:latest` is the tag of the docker container to run. I'm just using the latest container here, but if a specific version of MongoDB is required, you can find the tags in the [list of official MongoDB tags on DockerHub](https://hub.docker.com/_/mongo/tags). The `27017` is the default port used by MongoDB.

Containers that are declared as `static` will be shared between the test methods, and started only once, which will last through all the test methods, then it will be shut down. Containers declared as `instance` fields will be started and stopped for every test method.

For connecting to the database in the container, we still need our usual database connection settings in our `application.properties` in the `src/test/resources` folder. But as the port exposed by our docker might not be the same every time, we need to make it a variable, and get the value from the `@Container`. Here, we specify the `${mongodb.container.port}` as the variable.

```
spring.data.mongodb.database=OAuth2Sample
spring.data.mongodb.port=${mongodb.container.port}
spring.data.mongodb.host=localhost
spring.data.mongodb.auto-index-creation=true
```

Then we get the mapped port from the container and set it in the variable. And in order to reuse it, we make it a configuration class. 

```
@Configuration
@EnableMongoRepositories
public class MongoDBTestContainerConfig {
    @Container
    public static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo:latest")
            .withExposedPorts(27017);

    static {
        mongoDBContainer.start();
        var mappedPort = mongoDBContainer.getMappedPort(27017);
        System.setProperty("mongodb.container.port", String.valueOf(mappedPort));
    }
}
```

And when we need to use it in our test class, we add this configuration class to the `@ContextConfiguration` annotation.

```
@DataMongoTest
@Testcontainers
@ContextConfiguration(classes = MongoDBTestContainerConfig.class)
class AppUserRepositoryTest {

    @Autowired
    MongoTemplate mongoTemplate;

    @Autowired
    AppUserRepository appUserRepository;

    @Test
    void givenUserExists_whenFindByUsername_thenGetUser() {
        AppUser appUser = new AppUser("user1", "password");
        mongoTemplate.save(appUser);

        Optional<AppUser> user = appUserRepository.findByUsername("user1");
        assertTrue(user.isPresent());
        assertThat(user.get().getUsername(), is("user1"));
        assertThat(user.get().getPassword(), is("password"));
    }

    @Test
    void givenNonExistingUser_whenFindByUsername_thenReturnNotPresent(){
        Optional<AppUser> user = appUserRepository.findByUsername("something");
        assertTrue(user.isEmpty());
    }
}
```

A working example of the above code is available on [https://github.com/thecodinganalyst/oauth2](https://github.com/thecodinganalyst/oauth2). This is part of a project to demonstrate the [implementation of OpenID Connect / OAuth2 for login](https://www.thecodinganalyst.com/tutorial/OAuth2-login-with-spring-boot/), and [creating of a user in MongoDB for users who logged in for the first time](https://www.thecodinganalyst.com/tutorial/how-to-create-a-user-in-spring-after-login-with-openid-connect/).

