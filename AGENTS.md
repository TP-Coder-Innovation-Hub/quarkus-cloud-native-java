# AGENTS.md

Context and guidelines for AI coding assistants working in this Quarkus Cloud-Native Java learning path repository.

## Context

This repository contains educational content and hands-on code modules for learning Quarkus, a cloud-native Java framework. The content targets Java developers transitioning from traditional Java (Spring Boot, Jakarta EE) to cloud-native development with Quarkus 3.x. All content must be technically accurate for Quarkus 3.x as of 2026.

## Audience

- Java developers with 1--5+ years of experience.
- Familiar with core Java, Maven or Gradle, and basic Spring or Jakarta EE concepts.
- Learning cloud-native concepts: containers, Kubernetes, serverless, reactive programming.
- Skill levels: Entry (new to cloud-native), Mid (some experience), Senior (architectural decisions).

## How to Help

- Write Quarkus 3.x code using current APIs and defaults. RESTEasy Reactive is the default REST implementation (not classic RESTEasy). Use `@Path`, `@GET`, `@POST` JAX-RS annotations.
- Use Mutiny (`io.smallrye.mutiny.Uni`, `io.smallrye.mutiny.Multi`) for reactive code. Do not use Project Reactor or RxJava unless specifically discussing interoperability.
- Use Hibernate ORM with Panache (`PanacheEntity`, `PanacheRepository`) for database access patterns. Show both Active Record and Repository patterns where appropriate.
- For native image builds, assume GraalVM for JDK 21 or Mandrel. Use `@RegisterForReflection` for reflection registration in examples.
- Use Dev Services for test infrastructure. Do not require manual Docker Compose setup for examples.
- Include both JVM mode and native mode instructions where relevant.
- Use Maven as the primary build tool unless the user specifies Gradle. Standard Quarkus Maven project structure.
- Tag content difficulty with `[Entry]`, `[Mid]`, or `[Senior]` badges in Markdown headers or paragraphs.
- Provide complete, runnable code examples. No pseudocode or partial snippets that cannot compile.
- When explaining concepts, relate to Spring Boot equivalents where it helps comprehension (e.g., "Panache Entity is similar to Spring Data JPA's `JpaRepository`").
- Use Mermaid diagrams for architecture, flowcharts, and decision trees.
- Test instructions should use `mvn quarkus:dev` for dev mode and `mvn test` for running tests.

## How NOT to Help

- Do not use Spring Boot annotations (`@RestController`, `@RequestMapping`, `@Service`, `@Repository`, `@Value`, `@ComponentScan`) in Quarkus code examples. Use CDI and JAX-RS equivalents.
- Do not reference Quarkus 1.x or 2.x APIs. If uncertain about whether an API is current, verify against Quarkus 3.x documentation.
- Do not assume runtime classpath scanning or runtime bean discovery works the same as Spring. Quarkus resolves beans at build time.
- Do not use `spring-boot-starter-*` dependencies or Spring auto-configuration patterns.
- Do not recommend using Project Reactor (`Mono`, `Flux`) or RxJava as primary reactive types. Mutiny is the Quarkus standard.
- Do not provide native image build instructions without mentioning reflection constraints and `@RegisterForReflection`.
- Do not skip Dev Services when showing test examples. Tests should leverage automatic container provisioning.
- Do not use emojis in documentation or code comments. This repository uses plain text formatting only.
- Do not generate placeholder content, TODO comments, or stub implementations. All code must be complete and functional.
- Do not mix reactive and imperative paradigms in a single code example unless explicitly demonstrating interoperability.

## Key Concepts

- **Build-time processing:** Quarkus shifts annotation scanning, bean discovery, route registration, and configuration resolution from runtime to build time. This is the core architectural difference from traditional Java frameworks.
- **RESTEasy Reactive:** Default REST layer in Quarkus 3.x. Uses JAX-RS annotations but runs on Vert.x. Supports both blocking (worker threads) and non-blocking (event loop) execution models.
- **Mutiny:** Event-driven reactive programming library. `Uni<T>` for single-result async operations, `Multi<T>` for streams. Uses `.onItem()`, `.onFailure()`, `.onItem().transform()` chains.
- **Panache:** ORM abstraction over Hibernate. Active Record pattern (`extends PanacheEntity`) or Repository pattern (`extends PanacheRepository<T>`). Simplifies common CRUD operations.
- **GraalVM Native Image:** Ahead-of-time compilation to standalone native executable. Requires reflection registration via `@RegisterForReflection` or extension-provided metadata. Use Mandrel for Quarkus-optimized builds.
- **Dev Services:** Automatic container provisioning for development and testing. PostgreSQL, Kafka, Redis, Keycloak, and many other services start automatically in dev and test modes.
- **ArC:** Quarkus's CDI implementation. Build-time bean discovery. Uses `@ApplicationScoped`, `@RequestScoped`, `@Dependent`, `@SessionScoped`. No runtime component scanning.
- **Quarkus Extensions:** Modular add-ons that integrate libraries with Quarkus's build-time processing. Each extension provides annotation processing, native image metadata, and Dev Services integration. Searchable at `https://code.quarkus.io/`.

## Quarkus Guidelines 2026

- **Quarkus version:** 3.x (latest stable). RESTEasy Reactive is default. Classic RESTEasy is legacy.
- **Java version:** JDK 21 (LTS). GraalVM for JDK 21 or Mandrel for native compilation.
- **Build tools:** Maven (primary) or Gradle. Use Quarkus Maven plugin for dev mode (`quarkus:dev`) and native builds (`-Dnative`).
- **Configuration:** `application.properties` (primary) or `application.yml`. Use `@ConfigProperty` for injection, not `@Value`.
- **Testing:** JUnit 5 with Quarkus Test (`@QuarkusTest`). Use `@TestHTTPResource` or `RestAssured` for endpoint testing. Dev Services provide test databases automatically.
- **Dependency injection:** CDI-based. Use `@Inject`, `@Produces`, `@ApplicationScoped`. No `@Autowired`, no `@Component`.
- **REST:** JAX-RS annotations. `@Path`, `@GET`, `@POST`, `@PUT`, `@DELETE`, `@Produces`, `@Consumes`. Use `Response` or return types directly.
- **Reactive:** Mutiny only. `Uni<T>` and `Multi<T>`. No Reactor, no RxJava as primary types.
- **Database:** Hibernate ORM + Panache. Use Dev Services for automatic PostgreSQL in dev/test.
- **Native builds:** `mvn package -Dnative` or `mvn package -Dnative -Dquarkus.native.container-build=true`. Always address reflection constraints.

## Repository Structure

```
quarkus-cloud-native-java/
├── README.md                  # This fundamentals guide
├── AGENTS.md                  # AI assistant guidelines (this file)
├── modules/
│   ├── 01-first-microservice/  # Project setup, REST, DI, config
│   ├── 02-data-access/         # Hibernate ORM, Panache, migrations
│   ├── 03-reactive/            # Mutiny, Uni, Multi, reactive streams
│   ├── 04-native-deployment/   # GraalVM, native image, Docker, K8s
│   ├── 05-observability/       # Health, metrics, tracing, resilience
│   └── 06-event-driven/        # Kafka, reactive messaging, event sourcing
└── exercises/
    ├── challenges/             # Practice exercises per module
    └── solutions/              # Reference solutions
```

Each module directory should contain:
- `README.md` -- Module guide with explanations, code examples, and exercises.
- `src/` -- Complete, runnable Quarkus project (Maven structure).
- `src/main/java/` -- Java source code.
- `src/main/resources/` -- `application.properties`, native image configs.
- `src/test/java/` -- Test classes using `@QuarkusTest`.
