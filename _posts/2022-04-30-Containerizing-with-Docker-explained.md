---
title: "Containerizing with Docker explained"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Docker
  - Containerization
  - Spring Boot
  - Kotlin
---

After you have created your application, it is always nice to have it containerized so that it can be readily used whenever you need it, without having to clone/download the code from git, and build it. Who knows there might be some error when you try to build or run it in the distant future, you might be having a newer version of java or even using a different operating system, such that you can't run your old code anymore. 

Containerising your application create a snapshot of your built and running application, together with all the necessary environment, like your java version etc, so that your application can be platform independent. You just start your docker to run the application, and you access it like accessing another computer.

To do that, simply create a file named `Dockerfile` in the main folder of your code, then write the instructions in the file to

1. Specify which base image you need to use. This is the 
2. Define the location of the working directory in your image
3. Copy the executable from your current code directory to the working directory in your image
4. Specify which port to expose
5. Specify the command to run your executable

An example to create a docker image of a spring boot kotlin application is as below

```
FROM adoptopenjdk/openjdk11:alpine-jre
WORKDIR /app
COPY build/libs/eule-0.0.1-SNAPSHOT.jar .
EXPOSE 8080
CMD ["java", "-jar", "eule-0.0.1-SNAPSHOT.jar"]
```

The words in capital case are keywords from the dockerfile reference, and you can find more information about these keywords from [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/).

The first line - `FROM adoptopenjdk/openjdk11:alpine-jre` instructs docker to use [adoptopenjdk 11](https://hub.docker.com/r/adoptopenjdk/openjdk11) image from the alpine linux OS as the base. I'm running a kotlin application, and kotlin applications will be compiled as java runtime. Since my application is based on java 11, so I'll need to find a java 11 environment as a base image so that I can execute my jar file. You can choose any other images available on [DockerHub](https://hub.docker.com/). Alpine is a very lightweight linux, and I chose it so that my container will be small. AdoptOpenJdk is just one of the numerous openjdk, you can also choose amazoncorretto, ibmjava, bitnami, etc. 

Next, the [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir) establishes which directory on your container to be your working directory to run your application. If the folder you specify doesn't exist, it will be created. So `WORKDIR /app` will create an `app` folder under root `/`, and it will be used as our working directory. 

The [`COPY`](https://docs.docker.com/engine/reference/builder/#copy) command copies the file from your application in your current folder into the container, in the format `COPY [source] [target]`. So in the above code, we copy the jar file from my applicatin to the current folder (denoted bu `.`) in the image, which is the working directory - `/app`. The location of your jar file and the name of your jar file will vary depending on your application, and of course you need to build the jar so that it exists before you can build your docker file. 

The [`EXPOSE 8080`](https://docs.docker.com/engine/reference/builder/#expose) only informs docker that the port 8080 will be exposed from the container, it does not actually publish the port. The port is published from your application. I am running a spring boot application, and 8080 is the default port. 

Lastly, the [`CMD`](https://docs.docker.com/engine/reference/builder/#cmd) command will tell docker to run the command as specified in the parameter. As in `CMD ["java", "-jar", "eule-0.0.1-SNAPSHOT.jar"]`, it accepts an array, of which the first parameter is the command, and the rest of the parameters are the parameters of the command. In this specific example, we tell Docker to run `java -jar eule-0.0.1-SNAPSHOT.jar`. 

That's it! Make sure you have DockerDesktop running on your machine, or download and install it from [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/). Then simply run the command `docker build -t eule .` to build the image. The `-t eule` instructs docker to tag the image to the name - eule. The last `.` specifies the Dockerfile is in the current directory. After the image is build, you should be able to see it in your docker desktop. For more information on the docker build command, visit - [https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/).

![docker desktop images tab](/assets/images/2022/04/docker-image.png)

Or if you prefer to use the command line, the command `docker images` will list the images available. 

```
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
eule         latest    ebf80fd62f9b   47 minutes ago   199MB
```

To run it, use the command `docker run -d -p 8000:8080 eule`. The `-d` instructs docker to run it in detached mode, so that your terminal will not be stuck until you kill the container. The `-p 8000:8080` instructs docker to publish the port 8080 from your image to be port 8000 on your local machine. So although the image exposes port 8080, you will access it from port 8000 from your local machine. This is so that you won't be stuck in case you are running multiple containers all exposing the same port or the port number is already in use on your local machine. 

Lastly, you don't have to store the image on your computer, like git, there is also a repository to store your docker images, and that is [DockerHub](https://hub.docker.com/), the same place where you get the images as your base. To store your image on dockerhub, you'll need to create an account, tag your image accordingly to your account name, then push the image. Your image should be named in the format [dockerhub account]/[image name]. So as my dockerhub account is `thecodinganalyst`, I will first run `docker image tag eule thecodinganalyst/eule`, then `docker push thecodinganalyst/eule`. That's it, you should be able to see your image in your repositories in your dockerhub. The next time when you want to run the application on another computer, simply run `docker run -d -p 9000:8080 thecodinganalyst/eule`, then docker will automatically download the image named `thecodinganalyst/eule` from docker hub before running it. 
