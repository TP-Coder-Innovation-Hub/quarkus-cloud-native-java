# Capstone: Build a Cloud-Native API

`[Mid]` `[Senior]`

Build a complete cloud-native product catalog API with GraalVM native compilation, containerization, health checks, metrics, and tracing.

## Requirements

The service exposes a REST API for managing products. It connects to PostgreSQL, validates input, and compiles to a native executable. It includes health checks, Prometheus metrics, and OpenTelemetry tracing.

## Step 1: Generate the Project

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:create \
    -DplatformVersion=3.21.0 \
    -DprojectGroupId=com.example \
    -DprojectArtifactId=catalog-service \
    -DclassName=com.example.ProductResource \
    -Dpath=/products \
    -Dextensions="resteasy-reactive,jdbc-postgresql,hibernate-orm-panache,flyway,hibernate-validator,smallrye-health,micrometer-registry-prometheus,opentelemetry,smallrye-fault-tolerance,container-image-docker"
```

## Step 2: Entity

```java
package com.example;

import io.quarkus.hibernate.orm.panache.PanacheEntity;
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Table;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "products")
public class Product extends PanacheEntity {

    @NotBlank
    @Column(nullable = false)
    public String name;

    @Column
    public String description;

    @NotNull
    @Column(nullable = false)
    public BigDecimal price;

    @Column
    public String category;

    @Column(name = "created_at", updatable = false)
    public LocalDateTime createdAt = LocalDateTime.now();
}
```

## Step 3: REST Resource

```java
package com.example;

import io.quarkus.panache.common.Sort;
import jakarta.inject.Inject;
import jakarta.transaction.Transactional;
import jakarta.validation.Valid;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;
import java.util.List;

@Path("/products")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ProductResource {

    @GET
    public List<Product> list(
            @QueryParam("category") String category,
            @QueryParam("page") @DefaultValue("0") int page,
            @QueryParam("size") @DefaultValue("20") int size) {
        if (category != null) {
            return Product.find("category", Sort.by("name"), category)
                .page(page, size).list();
        }
        return Product.findAll(Sort.by("name")).page(page, size).list();
    }

    @GET
    @Path("/{id}")
    public Product get(@PathParam("id") Long id) {
        Product product = Product.findById(id);
        if (product == null) {
            throw new NotFoundException("Product not found: " + id);
        }
        return product;
    }

    @POST
    @Transactional
    @Status(Response.Status.CREATED)
    public Product create(@Valid Product product) {
        product.persist();
        return product;
    }

    @PUT
    @Path("/{id}")
    @Transactional
    public Product update(@PathParam("id") Long id, @Valid Product updated) {
        Product product = Product.findById(id);
        if (product == null) {
            throw new NotFoundException("Product not found: " + id);
        }
        product.name = updated.name;
        product.description = updated.description;
        product.price = updated.price;
        product.category = updated.category;
        return product;
    }

    @DELETE
    @Path("/{id}")
    @Transactional
    @Status(Response.Status.NO_CONTENT)
    public void delete(@PathParam("id") Long id) {
        Product.deleteById(id);
    }
}
```

## Step 4: Database Migration

Create `src/main/resources/db/migration/V1.0.0__Create_products.sql`:

```sql
CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    description TEXT,
    price       DECIMAL(10,2) NOT NULL,
    category    VARCHAR(100),
    created_at  TIMESTAMP DEFAULT NOW()
);
```

## Step 5: Health Check

```java
package com.example;

import io.agroal.api.AgroalDataSource;
import io.quarkus.arc.Arc;
import io.smallrye.health.annotation.Readiness;
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;
import java.sql.Connection;
import java.sql.SQLException;

@Readiness
@ApplicationScoped
public class DatabaseHealthCheck implements HealthCheck {

    @Override
    public HealthCheckResponse call() {
        AgroalDataSource ds = Arc.container().instance(AgroalDataSource.class).get();
        try (Connection conn = ds.getConnection()) {
            return HealthCheckResponse.named("database")
                .status(conn.isValid(2))
                .build();
        } catch (SQLException e) {
            return HealthCheckResponse.named("database")
                .down()
                .withData("error", e.getMessage())
                .build();
        }
    }
}
```

## Step 6: Configuration

```properties
# src/main/resources/application.properties

# HTTP
quarkus.http.port=8080

# Database
quarkus.datasource.db-kind=postgresql
quarkus.hibernate-orm.database.generation=none
quarkus.flyway.migrate-at-start=true

# Dev Services (automatic PostgreSQL in dev/test)
%dev.quarkus.hibernate-orm.database.generation=drop-and-create
%dev.quarkus.flyway.migrate-at-start=false

# Health
quarkus.smallrye-health.root-path=q/health

# Tracing
quarkus.otel.exporter.otlp.endpoint=http://localhost:4317
quarkus.otel.resource.attributes=service.name=catalog-service

# Container
quarkus.container-image.name=catalog-service
quarkus.container-image.tag=1.0.0
```

## Step 7: Test

```java
package com.example;

import io.quarkus.test.junit.QuarkusTest;
import io.restassured.http.ContentType;
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.*;

@QuarkusTest
class ProductResourceTest {

    @Test
    void testCreateAndList() {
        given()
            .contentType(ContentType.JSON)
            .body("{\"name\":\"Widget\",\"price\":9.99,\"category\":\"tools\"}")
            .when().post("/products")
            .then()
            .statusCode(201)
            .body("name", is("Widget"))
            .body("id", notNullValue());

        given()
            .when().get("/products")
            .then()
            .statusCode(200)
            .body("size()", greaterThan(0));
    }

    @Test
    void testValidation() {
        given()
            .contentType(ContentType.JSON)
            .body("{\"price\":9.99}")
            .when().post("/products")
            .then()
            .statusCode(400);
    }

    @Test
    void testHealthEndpoint() {
        given()
            .when().get("/q/health/ready")
            .then()
            .statusCode(200)
            .body("status", is("UP"));
    }
}
```

```bash
# Run tests (Dev Services provides PostgreSQL automatically)
mvn test
```

## Step 8: Build Native Image and Container

```bash
# Build native executable in a container
mvn package -Dnative \
    -Dquarkus.native.container-build=true \
    -Dquarkus.container-image.build=true

# Run the container
docker run -p 8080:8080 \
    -e QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://host.docker.internal:5432/catalog \
    -e QUARKUS_DATASOURCE_USERNAME=postgres \
    -e QUARKUS_DATASOURCE_PASSWORD=postgres \
    catalog-service:1.0.0
```

## Step 9: Verify

```bash
# Check startup time (should be under 30ms native)
docker run --rm catalog-service:1.0.0 2>&1 | head -5

# Create a product
curl -X POST http://localhost:8080/products \
    -H "Content-Type: application/json" \
    -d '{"name":"Bolt","price":0.50,"category":"hardware"}'

# List products
curl http://localhost:8080/products

# Health check
curl http://localhost:8080/q/health/ready

# Metrics
curl http://localhost:8080/q/metrics
```

## What You Built

- REST API with validation, pagination, and error handling
- Database access with Panache and Flyway migrations
- Health checks for Kubernetes readiness/liveness probes
- Prometheus metrics endpoint
- OpenTelemetry tracing with automatic context propagation
- Native image compilation producing a container under 100 MB

Previous: [Observability](../02-native-and-cloud/04-observability.md)
