# Quarkus Cloud-Native Java

A hands-on learning path for building cloud-native applications with Quarkus 3.x, GraalVM native compilation, and reactive programming.

**Assumes Java fundamentals.** You should be comfortable with Maven, annotations, and basic Spring or Jakarta EE concepts.

---

## Learning Path

| # | Topic | Level | Description |
|---|-------|-------|-------------|
| 00 | [Foundations](00-foundations/) | Entry | Why the JVM struggles in the cloud and how Quarkus solves it |
| 01 | [Quarkus Core](01-quarkus-core/) | Entry -- Mid | Dev mode, REST APIs, database access, reactive programming, DI |
| 02 | [Native & Cloud](02-native-and-cloud/) | Mid -- Senior | GraalVM native image, containers, microservice patterns, observability |
| 03 | [Capstone](03-capstone/) | Mid -- Senior | Build and deploy a cloud-native API with native compilation |

### 00-foundations/

| File | Level | Content |
|------|-------|---------|
| [01-why-cloud-native-java.md](00-foundations/01-why-cloud-native-java.md) | Entry | JVM's cloud problem: slow startup, high memory. Why density matters |
| [02-quarkus-approach.md](00-foundations/02-quarkus-approach.md) | Entry | Build-time processing. What moves from runtime to build |

### 01-quarkus-core/

| File | Level | Content |
|------|-------|---------|
| [01-dev-mode.md](01-quarkus-core/01-dev-mode.md) | Entry | Live reload, continuous testing, dev services |
| [02-rest-api.md](01-quarkus-core/02-rest-api.md) | Entry | JAX-RS routing, validation, error handling |
| [03-database.md](01-quarkus-core/03-database.md) | Entry -- Mid | Panache ORM, entities, repositories, migrations |
| [04-reactive.md](01-quarkus-core/04-reactive.md) | Mid | Mutiny, Vert.x, decision framework |
| [05-dependency-injection.md](01-quarkus-core/05-dependency-injection.md) | Entry | CDI in Quarkus vs Spring DI |

### 02-native-and-cloud/

| File | Level | Content |
|------|-------|---------|
| [01-graalvm-native.md](02-native-and-cloud/01-graalvm-native.md) | Mid | Native image benefits, costs, when native vs JVM |
| [02-containerization.md](02-native-and-cloud/02-containerization.md) | Mid | Optimized Docker images, container builds |
| [03-microservice-patterns.md](02-native-and-cloud/03-microservice-patterns.md) | Mid -- Senior | MicroProfile fault tolerance, health, metrics |
| [04-observability.md](02-native-and-cloud/04-observability.md) | Senior | Metrics, tracing, health endpoints |

---

## Objectives

By the end of this learning path you will be able to:

1. Explain why traditional JVM frameworks are suboptimal for cloud-native deployments and how build-time processing solves this.
2. Build REST APIs with Quarkus using JAX-RS, Panache ORM, and CDI dependency injection.
3. Decide when to use reactive (Mutiny) vs imperative code based on concrete requirements.
4. Compile a Quarkus application to a GraalVM native image and deploy it as an optimized container.
5. Instrument a Quarkus service with health checks, metrics, and distributed tracing.

---

> Part of the TP-Coder Innovation Hub educational content.
