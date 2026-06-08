# Dependency Injection in Quarkus

`[Entry]`

Quarkus uses ArC, a CDI-based (Contexts and Dependency Injection) implementation with build-time bean resolution. This is fundamentally different from Spring's runtime component scanning.

## CDI vs Spring: Key Differences

| Concept | Spring | Quarkus CDI |
|---------|--------|-------------|
| Component annotation | `@Component`, `@Service`, `@Repository` | `@ApplicationScoped`, `@RequestScoped`, `@Dependent` |
| Injection | `@Autowired` | `@Inject` |
| Configuration | `@Value("${prop}")` | `@ConfigProperty(name = "prop")` |
| Bean discovery | Runtime classpath scanning | Build-time annotation processing |
| Producers | `@Bean` on methods | `@Produces` on methods or fields |
| Scopes | `@Scope("request")`, `@Scope("session")` | `@RequestScoped`, `@SessionScoped` |

## Step by Step

### 1. Define a Service

```java
@ApplicationScoped
public class GreetingService {

    public String greet(String name) {
        return "Hello, " + name + "!";
    }
}
```

`@ApplicationScoped` creates one instance for the application lifecycle. Equivalent to Spring's singleton scope.

### 2. Inject It

```java
@Path("/greet")
public class GreetingResource {

    @Inject
    GreetingService service;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello(@QueryParam("name") String name) {
        return service.greet(name);
    }
}
```

### 3. Configuration Injection

```properties
# application.properties
greeting.prefix=Hello
greeting.suffix=!
```

```java
@ApplicationScoped
public class GreetingService {

    @ConfigProperty(name = "greeting.prefix", defaultValue = "Hello")
    String prefix;

    @ConfigProperty(name = "greeting.suffix", defaultValue = "!")
    String suffix;

    public String greet(String name) {
        return prefix + ", " + name + suffix;
    }
}
```

### 4. Producer Methods

When you need to create beans that are not CDI-managed (third-party objects):

```java
@ApplicationScoped
public class HttpClientProducer {

    @Produces
    @ApplicationScoped
    ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        mapper.registerModule(new JavaTimeModule());
        return mapper;
    }
}
```

Inject anywhere:

```java
@Inject ObjectMapper mapper;
```

### 5. Scopes

```java
@ApplicationScoped
public class AppWideCache { }
// One instance for the entire application.

@RequestScoped
public class RequestContextData { }
// New instance per HTTP request. State is not shared.

@Dependent
public class HelperUtility { }
// New instance every time it is injected. Default scope if none specified.

@SessionScoped
public class UserSession { }
// One instance per HTTP session. Requires session tracking.
```

### 6. Qualifiers

When you have multiple implementations of the same interface:

```java
@Qualifier
@Retention(RUNTIME)
@Target({FIELD, METHOD, PARAMETER, TYPE})
public @interface Json {}

@Qualifier
@Retention(RUNTIME)
@Target({FIELD, METHOD, PARAMETER, TYPE})
public @interface Xml {}
```

```java
@ApplicationScoped
public class JsonFormatter implements Formatter {
    @Override
    public String format(Object obj) { /* ... */ }
}

@ApplicationScoped
public class XmlFormatter implements Formatter {
    @Override
    public String format(Object obj) { /* ... */ }
}
```

Wait -- CDI requires explicit binding. Use `@Produces` with qualifiers:

```java
@ApplicationScoped
public class FormatterProducer {

    @Produces
    @Json
    Formatter jsonFormatter = new JsonFormatter();

    @Produces
    @Xml
    Formatter xmlFormatter = new XmlFormatter();
}
```

```java
@Inject @Json Formatter jsonFormatter;
@Inject @Xml Formatter xmlFormatter;
```

## What You Cannot Do

- No runtime bean registration. All beans are resolved at build time.
- No `@ComponentScan`. Quarkus discovers beans from the classpath during build.
- No `@Autowired`. Use `@Inject`.
- No `@Value`. Use `@ConfigProperty`.

If you try Spring patterns, you will get deployment errors. This is by design -- build-time resolution catches misconfiguration before the application starts.

Previous: [Reactive Programming](04-reactive.md) | Next: [GraalVM Native](../02-native-and-cloud/01-graalvm-native.md)
