---
layout: post
title:  "Containerizing a Spring Boot Application"
date:   2020-05-17 18:11:00 -0700
categories: software-development rest-api java spring-boot docker
---
[spring-boot-docker]: https://spring.io/guides/gs/spring-boot-docker/
[docker]: https://www.docker.com/

[Docker][docker] is a way to package and deploy applications so that they have the exact configuration and dependencies bundled together in any environment, whether it's on your local machine, a data center, or the cloud. This greatly simplifies application development, testing, and deployment -- you can be sure that your application is the same across all environments. No more fighting over dependencies between your applications, or between dev, QA, and production.

I'm going to containerize the [Barkr Java REST API]({% post_url 2020-05-09-creating-a-java-rest-api-with-spring-boot %}) I developed in a previous blog post.

## Some Prerequisites

To test and run all of this, you'll need access to Docker. Typically you'll just install Docker Desktop, available from the [Docker][docker] website.

## Creating a Dockerfile

A Dockerfile specifies how to create a Docker image. A Docker image is a packaged-up application and its dependencies, ready for deployment in a Docker environment.

Our Dockerfile for the Spring Boot REST API is, at this point, ridiculously simple, because our API right now is ridiculously simple. In later posts we'll explore how to add or run other components, e.g., a database, alongside an application.

```
FROM openjdk:8-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

The first line specifies the base image which will be the starting point for our application image. `openjdk:8-jdk-alpine` is a popular and relatively lightweight Linux-based image with Java JDK support.

The next 2 lines create a Linux user that will run the application. This helps mitigate some security risks, as by default an application will run with root privileges.

The last 3 lines tell Docker how to assemble and start our REST API. We only need to copy the jar file artifact that contains our API to the image, and invoke Java to run it.

## Building the Image

First, build the artifact:

```bash
$ ./mvnw clean compile package
# optionally test your package
$ java -jar target/barkr-api-0.0.1-SNAPSHOT.jar
```

Next, build the image:

```bash
$ docker build -t examplecom/barkr-api-docker .

Sending build context to Docker daemon  17.98MB
Step 1/6 : FROM openjdk:8-jdk-alpine
8-jdk-alpine: Pulling from library/openjdk
e7c96db7181b: Pull complete
f910a506b6cb: Pull complete
c2274a1a0e27: Pull complete
Digest: sha256:94792824df2df33402f201713f932b58cb9de94a0cd524164a0f2283343547b3
Status: Downloaded newer image for openjdk:8-jdk-alpine
 ---> a3562aa0b991
Step 2/6 : RUN addgroup -S spring && adduser -S spring -G spring
 ---> Running in a70634ea89fd
Removing intermediate container a70634ea89fd
 ---> 901028628a42
Step 3/6 : USER spring:spring
 ---> Running in 5833946b3a91
Removing intermediate container 5833946b3a91
 ---> 24eb90db2fa2
Step 4/6 : ARG JAR_FILE=target/*.jar
 ---> Running in 651abfa52328
Removing intermediate container 651abfa52328
 ---> 536a0288708a
Step 5/6 : COPY ${JAR_FILE} app.jar
 ---> 8eea6f93c44c
Step 6/6 : ENTRYPOINT ["java","-jar","/app.jar"]
 ---> Running in 3aa58353146f
Removing intermediate container 3aa58353146f
 ---> 914cb3020eca
Successfully built 914cb3020eca
Successfully tagged examplecom/barkr-api-docker:latest
```

The `-t` argument specifies the tag that you will apply to the newly-built image.

Docker fetches dependencies and images you specified in the Dockerfile and builds the image. You can see all images using `docker image ls`:

```bash
$ docker image ls
REPOSITORY                    TAG                 IMAGE ID            CREATED              SIZE
examplecom/barkr-api-docker   latest              914cb3020eca        About a minute ago   122MB
docker/getting-started        latest              3c156928aeec        4 weeks ago          24.8MB
openjdk                       8-jdk-alpine        a3562aa0b991        12 months ago        105MB
```

The `openjdk` image is the Alpine JDK 8 base image from which we built our `barkr-api-docker` image.

## Run It

Now we can run the image using the `docker run` command:

```bash
$ docker run -p 8080:8080 examplecom/barkr-api-docker

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.6.RELEASE)

2020-05-19 15:54:10.329  INFO 1 --- [           main] c.example.barkr.api.BarkrApiApplication  : Starting BarkrApiApplication v0.0.1-SNAPSHOT on 99118b6b3ac5 with PID 1 (/app.jar started by spring in /)
2020-05-19 15:54:10.334  INFO 1 --- [           main] c.example.barkr.api.BarkrApiApplication  : No active profile set, falling back to default profiles: default
2020-05-19 15:54:11.632  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-05-19 15:54:11.660  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-05-19 15:54:11.661  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.33]
2020-05-19 15:54:11.740  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-05-19 15:54:11.741  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1344 ms
2020-05-19 15:54:11.953  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-05-19 15:54:12.191  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-05-19 15:54:12.194  INFO 1 --- [           main] c.example.barkr.api.BarkrApiApplication  : Started BarkrApiApplication in 2.46 seconds (JVM running for 3.053)
```

We specify the image tag `examplecom/barkr-api-docker` as the image we want to run, and we publish (expose) the container's port 8080 to port 8080 so we can access the API locally.

We need to tweak our `docker run` command a little bit so the image runs in the background, by adding the `-d` flag:

```bash
$ docker run -d -p 8080:8080 examplecom/barkr-api-docker
83f2b7c7e083c6fe8a02270f06834c213d657721784a7acf40425e1e5649c3ba
```
To check if the image is running, we use the `docker ps` command:

```bash
$ docker ps
CONTAINER ID        IMAGE                         COMMAND                  CREATED              STATUS              PORTS                    NAMES
83f2b7c7e083        examplecom/barkr-api-docker   "java -jar /app.jar"     About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   silly_colden
```

If you don't specify a name in the `docker run` command, Docker creates a random one for you, in our case `silly_colden`. You can reference the running container using this name or the container ID.

## Test It

Now we can re-run our API calls to verify it works:

```bash
$ cat src/test/resources/test_bark.json | \
    curl -X POST -H 'Content-Type: application/json' \
    --data-binary @- 'http://localhost:8080/barks'

{"id":"c32fc1ef-47e0-4406-a784-9d6b916fdd18","creation_ts":"2020-05-19 16:00:53","title":"Hello Bark","content":"This is my first Bark."}

$ curl "http://localhost:8080/barks/c32fc1ef-47e0-4406-a784-9d6b916fdd18"

{"id":"c32fc1ef-47e0-4406-a784-9d6b916fdd18","creation_ts":"2020-05-19 16:00:53","title":"Hello Bark","content":"This is my first Bark."}

$ curl "http://localhost:8080/barks/nonexistent"

{"timestamp":"2020-05-19T16:01:26.398+0000","status":404,"error":"Not Found","message":"404 NOT_FOUND","path":"/barks/nonexistent"}%
```

## Summary

Packaging up a simple Java Spring Boot REST API in a Docker image is incredibly easy. Once packaged up, this image can be confidently deployed across different environments, as well as the cloud.

Future posts will explore adding more sophisticated logic and depenedencies to our REST API, as well as cloud deployment.
