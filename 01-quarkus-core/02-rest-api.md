# Building REST APIs with RESTEasy Reactive

`[Entry]`

RESTEasy Reactive is the default REST implementation in Quarkus 3.x. Standard JAX-RS annotations on Vert.x. Supports both blocking and non-blocking execution.

## Step by Step: Build an API

### 1. Add the Extension

```bash
mvn quarkus:add-extension -Dextensions="resteasy-reactive"
```

### 2. Create a Resource

```java
@Path("/tasks")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class TaskResource {

    @Inject
    TaskService taskService;

    @GET
    public List<Task> listAll() {
        return taskService.findAll();
    }

    @GET
    @Path("/{id}")
    public Task getById(@PathParam("id") Long id) {
        Task task = taskService.findById(id);
        if (task == null) {
            throw new NotFoundException("Task not found: " + id);
        }
        return task;
    }

    @POST
    @Status(Response.Status.CREATED)
    public Task create(Task task) {
        return taskService.create(task);
    }

    @PUT
    @Path("/{id}")
    public Task update(@PathParam("id") Long id, Task task) {
        return taskService.update(id, task);
    }

    @DELETE
    @Path("/{id}")
    @Status(Response.Status.NO_CONTENT)
    public void delete(@PathParam("id") Long id) {
        taskService.delete(id);
    }
}
```

### 3. Validation

Add the Hibernate Validator extension:

```bash
mvn quarkus:add-extension -Dextensions="hibernate-validator"
```

Annotate your model:

```java
public class Task {

    @NotNull
    @Size(min = 1, max = 200)
    public String title;

    @Size(max = 1000)
    public String description;

    @NotNull
    public Boolean completed = false;
}
```

Validation is automatic. Invalid requests return 400 with error details.

### 4. Error Handling

```java
@Provider
public class ErrorMapper implements ExceptionMapper<NotFoundException> {

    @Override
    public Response toResponse(NotFoundException e) {
        ErrorResponse error = new ErrorResponse(
            Response.Status.NOT_FOUND.getStatusCode(),
            e.getMessage()
        );
        return Response.status(Response.Status.NOT_FOUND)
            .entity(error)
            .build();
    }
}

public record ErrorResponse(int status, String message) {}
```

### 5. Configuration

```properties
# application.properties
quarkus.http.port=8080
quarkus.http.root-path=/api

# CORS for frontend
quarkus.http.cors=true
quarkus.http.cors.origins=http://localhost:3000
```

### 6. Test the API

```java
@QuarkusTest
class TaskResourceTest {

    @Test
    void testCreateAndRetrieve() {
        Task created = given()
            .contentType(ContentType.JSON)
            .body("{\"title\":\"Learn Quarkus\"}")
            .when().post("/tasks")
            .then()
            .statusCode(201)
            .extract().as(Task.class);

        given()
            .when().get("/tasks/" + created.id)
            .then()
            .statusCode(200)
            .body("title", is("Learn Quarkus"));
    }
}
```

## Key Points

- Use `@Path` and JAX-RS annotations. Not Spring's `@RestController`.
- `@Status` sets the response status code directly on the method.
- `@Provider` registers exception mappers. Works like Spring's `@ControllerAdvice` but uses JAX-RS.
- Validation is automatic with Hibernate Validator. No manual checks needed.

Previous: [Dev Mode](01-dev-mode.md) | Next: [Database Access](03-database.md)
