---
layout: post
title: "Creating a Java REST API With Spring Boot"
date: 2020-05-09 16:29:00 -0700
categories: software-development java spring-boot
---
During a recent interview, I was given a take-home coding exercise to create a simple REST API providing some basic functionality. I won't go into the specifics of this particular interview exercise, but it did have my scrambling a little bit, as it had been a while since I created a Java-based web application from scratch. Fortunately, I had been working with Spring Boot recently, so I opted to create the API that way.

I'm going to try to recreate that exercise here in detail, and expand on the functionality step-by-step. I'm doing this primarily as practice for how to create a REST API or microservice quickly using Java/Spring. Hopefully this will be helpful to you (and to my future self).

Finally, this project will serve as the basis for future advanced topics (hopefully!).

## Barkr: A Hypothetical Canine Social Network

We're going to try to build Barkr, a hypothetical social network for dogs. Users ("Barkrs") can post Barks, which contain an ID (a unique identifier generated when a Bark is saved), a creation timestamp, a title, and some content. Users can retrieve Barks by their ID. We'll add additional APIs once we get the basic form down.

Our REST API then will support the following API calls:

`POST /barks` - save a Bark

`GET /barks/{bark_id}` - retrieve a Bark with the specified ID, or return HTTP 404 if we can't find a Bark with that ID.

Our Bark JSON for creating a new Bark looks like:

```json
{
	"title": "I Like Dog Bones",
	"content": "Dog bones are my favorite treat!"
}
```

When the Bark is created, an ID and creation timestamp are generated and stored with it. Then whe the Bark is returned via API, it looks like:

```json
{
	"id": "some-unique-id",
	"creation_ts": "2020-05-17 09:12:15",
	"title": "I Like Dog Bones",
	"content": "Dog bones are my favorite treat!"
}
```

## About Spring Boot

[Spring Boot](https://spring.io/projects/spring-boot) is an opinionated framework for quickly and easily creating standalone Spring-based applications. Spring Boot is designed so that, if you followed the many conventions provided by the framework, you can rapidly create a web application or REST API with minimal scaffolding code. If you needed or wanted custom behavior at any layer, you can swap out the standard components and behavior with your own.

There are certainly other frameworks that let you rapidly build web applications and REST APIs in Java, but Spring Boot looks to be one of the most popular, and provides enough hooks to add advanced functionality should you need it.

## Bootstrapping with Spring Initializr

Spring provides the [Spring Initializr](https://start.spring.io/) online tool for creating the scaffolding (the initial project directory structure, build files, and placeholder classes) for your project. Simply visit the Initializr website, specify your project parameters, then click Generate to download a zip file containing the project scaffolding. Unpack that zip file in your working directory and start writing your code.

![Spring Initializr example](/assets/spring-initializr-example.png)

## Creating the Controller

The controller class contains methods for handling our API requests. We annotate these methods so that the framework knows how to map a request to its respective handler.

Our controller looks more or less like this:

```java
@RestController
public class BarksController {

    private static final Log LOGGER = LogFactory.getLog(BarksController.class);

    // we'll store our barks (posts) in a hash map from generated ID to Bark object
    private Map<String, Bark> barks = new HashMap<>();

    /**
     * Handler for HTTP POST requests for saving Barks (posts).
     *
     * @param bark
     * @return
     */
    @PostMapping("/barks")
    public Bark saveBark(@RequestBody Bark bark) {
        LOGGER.info("About to save " + bark);
        Bark savedBark = saveBarkInternal(bark);
        if (null != savedBark) {
            return savedBark;
        } else {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST);
        }
    }

    /**
     * Handler for HTTP GET requests for fetching previously-stored Barks (posts).
     *
     * @param barkId
     * @return
     */
    @GetMapping("/barks/{barkId}")
    public Bark getBark(@PathVariable String barkId) {
        LOGGER.info("Fetching bark with ID " + barkId);
        Bark bark = getBarkInternal(barkId);
        if (null != bark) {
            return bark;
        } else {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND);
        }
    }

    /**
     * Saves the Bark - generate a UUID and use it as the key into the barks map.
     */
    private Bark saveBarkInternal(Bark bark) {
        if (null != bark) {
            Bark barkToSave = new Bark(UUID.randomUUID().toString(),
                    LocalDateTime.now(),
                    bark.getTitle(),
                    bark.getContent()
            );
            this.barks.put(barkToSave.getId(), barkToSave);
            return barkToSave;
        } else {
            return null;
        }
    }

    /**
     * Returns the Bark (post) from the underlying map with the specified ID, or null if no such Bark exists.
     *
     * @param barkId
     * @return
     */
    private Bark getBarkInternal(String barkId) {
        Bark bark = this.barks.get(barkId);
        return bark;
    }
}
```

Note that we've co-mingled the "business logic" (how to store and retrieve the Barks) within this controller class. The regular Java convention is to separate this into some Service class that encapsulates the business logic. We could then, for example, have this simple implementation that uses an in-memory hash map to store Barks, or a more complex implementation that stores Barks in some underlying database or document store. We'll explore refactoring the logic here in a later post, but for now we just want to stand up this API quickly.

## Adding the Resource Class

We need to create a class to represent the Bark resource. The Spring Boot framework will handle serializing and deserializing (converting to/from JSON) for us, but we need to tell it what it looks like.

Our Bark resource looks like this:

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Bark {

    private final String id;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @JsonProperty("creation_ts")
    private final LocalDateTime creationTs;
    private final String title;
    private final String content;

    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    public Bark(
            @JsonProperty("id") String id,
            @JsonProperty("creation_ts") LocalDateTime creationTs,
            @JsonProperty("title") String title,
            @JsonProperty("content") String content) {
        this.id = id;
        this.creationTs = creationTs;
        this.title = title;
        this.content = content;
    }

    public String getId() {
        return this.id;
    }

    public LocalDateTime getCreationTs() {
        return this.creationTs;
    }

    public String getTitle() {
        return this.title;
    }

    public String getContent() {
        return this.content;
    }
}
```

A few interesting things:

* The `@JsonInclude(JsonInclude.Include.NON_NULL)` tells Spring (which uses Jackson JSON under the hood by default for JSON handling) to include only non-null values when performing serialization/deserialization.
* The `@JsonCreator(mode = JsonCreator.Mode.PROPERTIES)` annotation above the `Bark` constructor tells the Jackson JSON deserializer to use that constructor when creating objects from JSON. By default, the deserializer looks for setter methods, and here we've opted to create immutable objects (note the `private final` fields). The `@JsonProperty` annotations then inform the deserializer which field maps to which constructor argument.
* The `@JsonFormat` annotation above the `LocalDateTime` field tells Jackson the date format to use when serializing and deserializing dates.

## A Quick Note on the Spring Boot Application Class

Spring Initializr automatically generates an application class containing a `main()` method that is used to kick off the application. You typically don't need to worry about it unless you need to execute some specialized startup code, so I won't go into it now, but it looks like this:

```java
@SpringBootApplication
public class BarkrApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(BarkrApiApplication.class, args);
    }

}
```

## Running It

With our initial code all written, we can now run the service. At the project's root directory, invoke the following:

```bash
$ ./mvnw spring-boot:run
```

You should see output similar to the following:

```bash
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------------< com.example:barkr-api >------------------------
[INFO] Building barkr-api 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] >>> spring-boot-maven-plugin:2.2.6.RELEASE:run (default-cli) > test-compile @ barkr-api >>>
[INFO]
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ barkr-api ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ barkr-api ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ barkr-api ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ barkr-api ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] <<< spring-boot-maven-plugin:2.2.6.RELEASE:run (default-cli) < test-compile @ barkr-api <<<
[INFO]
[INFO]
[INFO] --- spring-boot-maven-plugin:2.2.6.RELEASE:run (default-cli) @ barkr-api ---
[INFO] Attaching agents: []

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.6.RELEASE)

2020-05-17 17:45:50.430  INFO 3224 --- [           main] c.example.barkr.api.BarkrApiApplication  : Starting BarkrApiApplication on raziel with PID 3224 (/Users/jquiaot/projects/barkr-rest-api/java/barkr-api/target/classes started by jquiaot in /Users/jquiaot/projects/barkr-rest-api/java/barkr-api)
2020-05-17 17:45:50.433  INFO 3224 --- [           main] c.example.barkr.api.BarkrApiApplication  : No active profile set, falling back to default profiles: default
2020-05-17 17:45:51.365  INFO 3224 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-05-17 17:45:51.377  INFO 3224 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-05-17 17:45:51.377  INFO 3224 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.33]
2020-05-17 17:45:51.474  INFO 3224 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-05-17 17:45:51.474  INFO 3224 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 989 ms
2020-05-17 17:45:51.618  INFO 3224 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-05-17 17:45:51.776  INFO 3224 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-05-17 17:45:51.779  INFO 3224 --- [           main] c.example.barkr.api.BarkrApiApplication  : Started BarkrApiApplication in 1.73 seconds (JVM running for 2.122)
```

Let's issue a few API calls to see if it works:

```bash
$ cat src/test/resources/test_bark.json | \
    curl -X POST -H 'Content-Type: application/json' \
    --data-binary @- 'http://localhost:8080/barks'

{"id":"d304d1ae-36ec-4fff-9dea-9c1f16c1619e","creation_ts":"2020-05-17 17:48:06","title":"Hello Bark","content":"This is my first Bark."}
```

```bash
$ curl "http://localhost:8080/barks/d304d1ae-36ec-4fff-9dea-9c1f16c1619e"

{"id":"d304d1ae-36ec-4fff-9dea-9c1f16c1619e","creation_ts":"2020-05-17 17:48:06","title":"Hello Bark","content":"This is my first Bark."}
```

```bash
$ curl "http://localhost:8080/barks/nonexistent"

{"timestamp":"2020-05-18T00:49:43.329+0000","status":404,"error":"Not Found","message":"404 NOT_FOUND","path":"/barks/nonexistent"}
```

Note that we haven't yet customized our HTTP 404 error message, so we get this output when we try to `GET` a non-existent Bark.

## Summary and References

In a short amount of time, we were able to get a Java-based Spring Boot REST API up and running.

A few online guides were useful in figuring out how to put this together:

* [Spring Boot][spring-boot-home]
* [Spring Initializr][spring-boot-init]
* [Spring Boot: Building a RESTful Web Service][spring-boot-rest]
* [Baeldung: Deserialize Immutable Objects with Jackson][baeldung-deserialize]


[spring-boot-home]: https://spring.io/projects/spring-boot
[spring-boot-init]: https://start.spring.io/
[spring-boot-rest]: https://spring.io/guides/gs/rest-service/
[baeldung-deserialize]: https://www.baeldung.com/jackson-deserialize-immutable-objects
