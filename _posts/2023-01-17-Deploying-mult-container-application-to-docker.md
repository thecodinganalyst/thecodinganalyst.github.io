---
title: "Deploying multi-container application to Docker"
excerpt_separator: "<!--more-->"
categories:
  - Tutorial
tags:
  - Docker
  - Docker Compose
  - Multi Stage Build
  - MariaDB
---

In my last post - [Getting started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/), I demonstrated how to create a sample forum application with Spring Boot using MariaDB as the database. In this article, I shall demonstrate how to deploy that application to docker.

![docker multi container](/assets/images/2023/01/docker-multi-container.png)

To start, we create a file named [`Dockerfile`](https://github.com/thecodinganalyst/forum/blob/docker/Dockerfile) in the root directory. This file contains the instruction for Docker to do a [multi-stage build](https://docs.docker.com/build/building/multi-stage/) to build the application in a gradle container, then copy the output jar file to another container. 

```
FROM gradle:7.6-jdk17-alpine AS build
COPY --chown=gradle:gradle . /home/gradle/src
WORKDIR /home/gradle/src
RUN gradle build --no-daemon

FROM amazoncorretto:17-alpine3.16
WORKDIR /app
COPY --from=build /home/gradle/src/build/libs/*.jar ./forum.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "forum.jar"]
```
<!--more-->

The key to doing a multi-stage build is to name the builds. In the file above, we add the `AS build` in line 1, so that we can copy the file we need from the gradle container in the amazoncorretto container in the line `COPY --from=build ...`. Do refer to [Containerizing with Docker explained](https://thecodinganalyst.github.io/knowledgebase/Containerizing-with-Docker-explained/) on more about the Dockerfile.

As we need 2 containers - 1 for the application and another for the database, we will need to use [Docker Compose](https://docs.docker.com/compose/). So we shall create a `docker-compose.yml` in our root directory.

```
networks:
  forumNetwork:
    driver: bridge
services:
  database:
    image: mariadb:10.11-rc
    environment:
      MARIADB_ROOT_PASSWORD_HASH: "*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19"
      MARIADB_DATABASE: "forum"
      MARIADB_USER: "forumApp"
      MARIADB_PASSWORD_HASH: "*0A7A4E64514FC3F69614759689B0E6D5F9C286CD"
    ports:
      - "3306:3306"
    volumes:
      - C:\Mariadb\data:/var/lib/mysql
    networks:
      - forumNetwork
  web:
    build: .
    ports:
      - "8000:8080"
    networks:
      - forumNetwork
    depends_on:
      - database
    restart: on-failure
``` 

With the following configuration, we first create a [user defined bridge network](https://docs.docker.com/network/network-tutorial-standalone/#use-user-defined-bridge-networks) named `forumNetwork` for the 2 containers to connect to each other.

```
networks:
  forumNetwork:
    driver: bridge
```

Next, we can start to define the 2 services - our java application, which we will build it it from the `Dockerfile` we just defined; and a MariaDB database which we can get it from the [official MariaDB dockerhub](https://hub.docker.com/_/mariadb). Let's start by going through the configurations for the MariaDB container.

```
database:
    image: mariadb:10.11-rc
    environment:
      MARIADB_ROOT_PASSWORD_HASH: "*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19"
      MARIADB_DATABASE: "forum"
      MARIADB_USER: "forumApp"
      MARIADB_PASSWORD_HASH: "*0A7A4E64514FC3F69614759689B0E6D5F9C286CD"
    ports:
      - "3306:3306"
    volumes:
      - C:\Mariadb\data:/var/lib/mysql
    networks:
      - forumNetwork
```

The `image: mariadb:10.11-rc` specifies the 10.11 release candidate version we wanted. Then we create the environment variables as described in the official MariaDB dockerhub page. The root password is mandatory, and we have to use either one of the following configs to specify it - `MARIADB_RANDOM_ROOT_PASSWORD`, `MARIADB_ROOT_PASSWORD_HASH`, `MARIADB_ROOT_PASSWORD`, `MARIADB_ALLOW_EMPTY_ROOT_PASSWORD`. 

We added the `MARIADB_DATABASE`, `MARIADB_USER`, and `MARIADB_PASSWORD_HASH` configs so that we can specify the database and user credentials we need for our application. The `ports` config specifies the host to container mapping for the port we are going to publish. 

The `volumes` config allows us to specify a location on our host computer where we can persist the data in our container, so that when we stop the container and restart it next time, the data we previously added will not be gone. In the config 

```
volumes:
   - C:\Mariadb\data:/var/lib/mysql
```

we map the location `C:\Mariadb\data` on our host computer to the location `/var/lib/mysql` in the mariadb container. Be sure to create the `C:\Mariadb\data` on your computer before we run the file. 

Lastly, the `networks` config is set to the user defined network name we created earlier.

Let us now define our application, which we will name it `web`.

```
web:
    build: .
    ports:
      - "8000:8080"
    networks:
      - forumNetwork
    depends_on:
      - database
    restart: on-failure
```

The `build: .` tells docker that we will build the image from the current location, denoted by the `.`. The `ports: - "8000:8080"` maps the host port `8000` to the container port `8080`. That means, when we want to access the application from our computer, we go to port 8000 - `http://localhost:8000`, even though in our application in the container, it defaults expose as port `8080`. The `networks` config again specifies the user defined network we created earlier.

As the application is dependent on the database to store and get all its data, we added the [`depends_on`](https://docs.docker.com/compose/compose-file/#depends_on) to specify that it needs the `database` container. However, during startup, the database container can be created, satisfying the `depends_on` criteria, but the database is not ready, as in the user or database is not created yet; but the application is already trying to connect to it, and it will fail. So we need to add the [`restart: on-failure`](https://docs.docker.com/compose/compose-file/#restart) to get the java container to try again when there's a failure. 

Last but not least, we need to update our [`application.properties`](https://github.com/thecodinganalyst/forum/blob/master/src/main/resources/application.properties) in our `src/main/resources` folder to point to the database in the container instead of local.

```
spring.datasource.url=jdbc:mariadb://localhost:3306/forum
``` 

to become 

```
spring.datasource.url=jdbc:mariadb://database:3306/forum
```

The `database` is the name of the database container. Inter-container reference can use the format `<protocol>://<container-name>:<container-port>`. Do note that for inter container networking, we use the container port instead of the host port. So if our port config for the mariadb container is `3006:3306`, we should still use `3306`. The host port `3006` is for when we want to connect from our computer. 

To finish it up, we run `docker compose up` in the root folder. Opening up the docker desktop, we can see a grouped container for our forum app, with 2 containers running.

![docker desktop](/assets/images/2023/01/docker-forum.png)

The source code for this sample is available in the `docker` branch of the repository - [https://github.com/thecodinganalyst/forum/tree/docker](https://github.com/thecodinganalyst/forum/tree/docker).

This is part of a series illustrating how to build a backend Spring boot application.
- [Getting Started Spring Boot Application](https://thecodinganalyst.github.io/tutorial/Spring-boot-application-getting-started/)
- [Deploying to Docker](https://thecodinganalyst.github.io/tutorial/Deploying-mult-container-application-to-docker/)
- [Spring Data Testing](https://thecodinganalyst.github.io/tutorial/how-to-test-spring-data-repository/)
- [Testing Services](https://thecodinganalyst.github.io/tutorial/how-to-test-services-in-a-spring-boot-application/)
- [Unit Testing of Controller](https://thecodinganalyst.github.io/tutorial/how-to-unit-test-rest-controller-in-a-spring-boot-application/)
- [Integration Testing](https://thecodinganalyst.github.io/knowledgebase/how-to-do-integration-testing-in-spring-boot-rest-application/)
- [Code quality review with Sonarqube](https://www.thecodinganalyst.com/tutorial/integrate-code-quality-review-with-sonarqube/)
- [Configure Spring Security CSRF for testing on Swagger](https://www.thecodinganalyst.com/tutorial/Configure-spring-security-csrf-for-testing-on-swagger/)
- [Configure Access Management in Spring Security](https://www.thecodinganalyst.com/tutorial/how-to-configure-access-management-in-spring-security/)