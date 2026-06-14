# AGENTS.md

Context and guidelines for AI coding assistants working in this repository.

## Context

Educational content for learning Quarkus, a cloud-native Java framework. Targets Java developers transitioning from Spring Boot or Jakarta EE. All content must be accurate for Quarkus 3.x as of 2026.

## Audience

- Java developers with 1--5+ years experience.
- Familiar with core Java, Maven, and basic Spring or Jakarta EE concepts.
- Skill levels: `` (new to cloud-native), `` (some experience), `` (architectural decisions).

## How to Help

- Write Quarkus 3.x code. RESTEasy Reactive is the default REST layer. Use JAX-RS annotations (`@Path`, `@GET`, `@POST`).
- Use Mutiny (`Uni`, `Multi`) for reactive code. Not Reactor or RxJava.
- Use Hibernate ORM with Panache for database access. Show both Active Record and Repository patterns.
- For native builds, assume GraalVM for JDK 21 or Mandrel. Use `@RegisterForReflection` for reflection.
- Use Dev Services for test infrastructure. No manual Docker Compose.
- Use Maven as primary build tool.
- Tag content with ``, ``, or `` badges.
- Provide complete, runnable code. No pseudocode or partial snippets.
- Relate to Spring Boot equivalents when it helps comprehension.
- Use Mermaid diagrams for architecture and decision trees.

## How NOT to Help

- Do not use Spring annotations (`@RestController`, `@RequestMapping`, `@Service`, `@Value`, `@ComponentScan`).
- Do not reference Quarkus 1.x or 2.x APIs.
- Do not assume runtime classpath scanning works like Spring. Quarkus resolves beans at build time.
- Do not recommend Reactor (`Mono`, `Flux`) or RxJava as primary reactive types.
- Do not provide native image instructions without mentioning reflection constraints.
- Do not skip Dev Services in test examples.
- Do not use emojis.
- Do not generate placeholder content, TODO comments, or stubs.
- Do not mix reactive and imperative paradigms unless demonstrating interoperability.

## Key Concepts

- **Build-time processing:** Annotation scanning, bean discovery, route registration, config resolution happen at build time.
- **RESTEasy Reactive:** Default REST layer. JAX-RS annotations on Vert.x. Supports blocking and non-blocking.
- **Mutiny:** `Uni<T>` for single async results, `Multi<T>` for streams. Chains: `.onItem()`, `.onFailure()`, `.onItem().transform()`.
- **Panache:** ORM over Hibernate. Active Record (`extends PanacheEntity`) or Repository (`extends PanacheRepository<T>`).
- **GraalVM Native Image:** AOT compilation. Requires `@RegisterForReflection` or extension metadata.
- **Dev Services:** Auto container provisioning for dev/test. PostgreSQL, Kafka, Redis, Keycloak, etc.
- **ArC:** Quarkus CDI implementation. Build-time bean discovery. Scopes: `@ApplicationScoped`, `@RequestScoped`, `@Dependent`.

## Quarkus Guidelines 2026

- **Version:** 3.x (latest stable). RESTEasy Reactive is default.
- **Java:** JDK 21 (LTS). GraalVM for JDK 21 or Mandrel.
- **Build:** Maven primary. `quarkus:dev` for dev mode, `-Dnative` for native builds.
- **Config:** `application.properties` or `application.yml`. `@ConfigProperty` for injection.
- **Testing:** JUnit 5, `@QuarkusTest`, RestAssured. Dev Services for test databases.
- **DI:** CDI. `@Inject`, `@Produces`, `@ApplicationScoped`. No `@Autowired`, no `@Component`.
- **REST:** JAX-RS. `@Path`, `@GET`, `@POST`, `@PUT`, `@DELETE`, `@Produces`, `@Consumes`.
- **Reactive:** Mutiny only. `Uni<T>` and `Multi<T>`.
- **Database:** Hibernate ORM + Panache. Dev Services for PostgreSQL.
- **Native:** `mvn package -Dnative` or `-Dquarkus.native.container-build=true`.

## Repository Structure

```
quarkus-cloud-native-java/
├── README.md
├── AGENTS.md
├── 00-foundations/
│   ├── 01-why-cloud-native-java.md
│   └── 02-quarkus-approach.md
├── 01-quarkus-core/
│   ├── 01-dev-mode.md
│   ├── 02-rest-api.md
│   ├── 03-database.md
│   ├── 04-reactive.md
│   └── 05-dependency-injection.md
├── 02-native-and-cloud/
│   ├── 01-graalvm-native.md
│   ├── 02-containerization.md
│   ├── 03-microservice-patterns.md
│   └── 04-observability.md
└── 03-workshop/
    └── README.md
```
