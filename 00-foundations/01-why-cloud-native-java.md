# Why Cloud-Native Java

`[Entry]` `[Mid]`

## The JVM's Cloud Problem

Cloud platforms -- Kubernetes, serverless, auto-scaling groups -- demand fast startup and low memory at rest. The traditional JVM was designed for neither.

A typical Spring Boot application in a Kubernetes pod:

- **Startup:** 2--5 seconds on a warm JVM. 10--15 seconds with classpath scanning and bean initialization on larger apps.
- **Memory:** 200--500 MB RSS for a service doing almost nothing. The JVM loads a GC heap, JIT compiler, and class metadata.
- **Scaling latency:** When a horizontal pod autoscaler fires, each new pod pays that startup cost. Users feel timeouts.

Cloud platforms bill by memory-seconds and CPU-seconds. A service idling at 300 MB serving 10 requests per minute wastes money. A service taking 12 seconds to start cannot respond to traffic spikes in time.

The JVM's design assumptions -- long-running processes, JIT warm-up periods, generous heap allocation -- are at odds with ephemeral, auto-scaling container workloads.

## Why Density Matters

Consider a Kubernetes cluster with 100 microservices. Each running Spring Boot at 300 MB idle:

- Total memory: 30 GB just for idle services.
- With Quarkus native at 50 MB each: 5 GB total.
- Savings: 25 GB of cluster memory. That is fewer nodes, lower cost, or room for more services.

Languages like Go and Rust start in milliseconds and consume tens of megabytes. They became the default for cloud-native tooling. Java had to change to stay relevant.

## How Java Responded

Three approaches emerged:

**GraalVM native image** -- Compiles Java bytecode ahead-of-time into a standalone native executable. No JVM startup, no JIT warm-up, no classpath scanning at runtime.

**Build-time processing** -- Moves framework work (annotation scanning, bean discovery, config resolution) from startup to the build phase. The running application starts with a pre-computed model.

**Reactive runtimes** -- Replaces thread-per-request with event-loop architectures. More concurrency with fewer threads and less memory.

Quarkus combines all three into a unified framework. The results:

| Metric | Traditional JVM | Quarkus JVM | Quarkus Native |
|--------|----------------|-------------|----------------|
| Startup | 2--15s | 0.5--1.5s | 10--30ms |
| Memory (idle) | 200--500 MB | 50--120 MB | 30--80 MB |

This moves Java from "too heavy for serverless" to "competitive with Go" on cloud-native metrics.

Next: [The Quarkus Approach](02-quarkus-approach.md)
