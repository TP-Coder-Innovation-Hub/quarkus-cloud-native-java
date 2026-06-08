# Dev Mode: The Developer Experience

`[Entry]`

Quarkus dev mode (`mvn quarkus:dev`) is not hot-redeploy bolted onto a framework. It is a continuous build-and-reload loop integrated with the build-time processing pipeline.

## Step by Step: Create and Run

```bash
# 1. Generate a new project
mvn io.quarkus.platform:quarkus-maven-plugin:create \
    -DplatformVersion=3.21.0 \
    -DprojectGroupId=com.example \
    -DprojectArtifactId=hello-quarkus \
    -DclassName=com.example.GreetingResource \
    -Dpath=/hello

# 2. Enter the project
cd hello-quarkus

# 3. Start dev mode
mvn quarkus:dev
```

The application starts in under a second. You see:

```
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
---
Listening for transport dt_socket at address: 5005
2026-01-15 10:00:00 INFO  [io.quarkus] (main) Installed features: [cdi, rest]
Tests paused, press [r] to resume, [h] for more options>
```

## Live Reload

Modify the generated `GreetingResource.java`:

```java
@Path("/hello")
public class GreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "Hello from Quarkus!";
    }
}
```

Save the file. The application reflects changes in under 100ms. No restart. Works for:

- Adding new REST endpoints
- Changing `@ConfigProperty` values
- Adding or removing CDI beans and `@Inject` points
- Modifying JPA entity mappings

## Continuous Testing

Press `r` in the dev mode console to enable continuous testing. When you save a source file, affected tests run automatically against the running application. Results appear in the console and Dev UI.

Create a test:

```java
@QuarkusTest
class GreetingResourceTest {

    @Test
    void testHelloEndpoint() {
        given()
            .when().get("/hello")
            .then()
            .statusCode(200)
            .body(is("Hello from Quarkus!"));
    }
}
```

Save it. The test runs. No manual `mvn test` invocation.

## Dev Services

When you add an extension that requires an external service and do not configure a connection URL, Quarkus starts a container automatically.

Add a PostgreSQL dependency:

```bash
mvn quarkus:add-extension -Dextensions="jdbc-postgresql,hibernate-orm"
```

In `application.properties`:

```properties
quarkus.hibernate-orm.database.generation=drop-and-create
```

Run `mvn quarkus:dev`. A PostgreSQL container starts, the app connects to it, and the container stops when dev mode exits. No Docker Compose. No manual setup.

## Comparison with Traditional Development

| Aspect | Traditional (Spring Boot) | Quarkus Dev Mode |
|--------|--------------------------|-------------------|
| Code change to feedback | 5--30 seconds | Under 1 second |
| Test execution | Manual (`mvn test`) | Continuous on save |
| External services | Docker Compose or manual | Automatic Dev Services |
| Config changes | Restart required | Live reload |
| Bean wiring changes | Restart required | Live reload |

Previous: [Quarkus Approach](../00-foundations/02-quarkus-approach.md) | Next: [REST API](02-rest-api.md)
