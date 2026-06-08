# Observability

``

Production Quarkus services need three things: metrics to know what is happening, traces to know where time is spent, and health endpoints to tell the platform if the service is alive.

## Metrics with Micrometer

### Step 1: Add Extension

```bash
mvn quarkus:add-extension -Dextensions="micrometer-registry-prometheus"
```

### Step 2: Built-in Metrics

Quarkus exposes JVM metrics, HTTP request metrics, and Vert.x event loop metrics automatically. No code required.

```
# Example built-in metrics
http_server_requests_seconds_count{method="GET",uri="/api/orders",status="200"} 1542
http_server_requests_seconds_sum{method="GET",uri="/api/orders",status="200"} 12.543
jvm_memory_used_bytes{area="heap"} 67108864
vertx_event_loop_execution_seconds_count 89012
```

### Step 3: Custom Metrics

```java
@ApplicationScoped
public class InventoryService {

    @Inject
    MeterRegistry registry;

    @Counted(value = "inventory.checks", description = "Number of inventory checks")
    public int getStockLevel(String sku) {
        return queryDatabase(sku);
    }

    public void updateStock(String sku, int quantity) {
        registry.gauge("inventory.stock.level",
            Tags.of("sku", sku),
            queryDatabase(sku));
        performUpdate(sku, quantity);
    }
}
```

### Step 4: Scrape

```
GET /q/metrics
```

Point Prometheus at this endpoint. Done.

## Distributed Tracing with OpenTelemetry

### Step 1: Add Extension

```bash
mvn quarkus:add-extension -Dextensions="opentelemetry"
```

### Step 2: Configure Exporter

```properties
# application.properties
quarkus.otel.exporter.otlp.endpoint=http://otel-collector:4317
quarkus.otel.resource.attributes=service.name=order-service,env=prod
quarkus.otel.traces.sampler=parentbased_traceidratio
quarkus.otel.traces.sampler.arg=0.1
```

### Step 3: Automatic Instrumentation

Quarkus automatically creates spans for HTTP requests, REST client calls, and database queries. No code changes required.

### Step 4: Manual Spans

```java
@ApplicationScoped
public class OrderProcessor {

    @WithSpan("process-order")
    public Order processOrder(
            @SpanAttribute("order.id") String orderId,
            OrderRequest request) {
        validateOrder(request);
        reserveInventory(request);
        chargePayment(request);
        return confirmOrder(orderId, request);
    }

    @WithSpan("validate-order")
    void validateOrder(OrderRequest request) {
        // validation logic
    }
}
```

### Step 5: Context Propagation

When calling downstream services, trace context propagates automatically:

```java
@RegisterRestClient(configKey = "inventory-service")
public interface InventoryClient {

    @GET
    @Path("/api/inventory/{sku}")
    Uni<StockLevel> getStock(@PathParam("sku") String sku);
}
```

The `traceparent` header is injected automatically. No manual propagation code.

## Health Endpoints

Health checks are covered in [Microservice Patterns](03-microservice-patterns.md). Key endpoints:

```
GET /q/health/live     - Liveness: is the process alive?
GET /q/health/ready    - Readiness: connected to DB, messaging, etc.?
GET /q/health/started  - Startup: has init completed?
```

## The Complete Picture

```
[Request] --> [Quarkus Service]
                  |
                  +-- Metrics (Micrometer) --> Prometheus --> Grafana
                  +-- Traces (OpenTelemetry) --> OTel Collector --> Jaeger/Tempo
                  +-- Health (/q/health/*) --> Kubernetes probes
```

Configure all three and you have full production observability. Metrics tell you what happened. Traces tell you where time was spent. Health checks tell the platform whether to route traffic.

Previous: [Microservice Patterns](03-microservice-patterns.md) | Next: [Capstone](../03-capstone/README.md)
