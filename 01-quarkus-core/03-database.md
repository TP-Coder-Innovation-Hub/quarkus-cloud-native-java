# Database Access with Panache

`[Entry]` `[Mid]`

Panache is Quarkus's ORM layer over Hibernate. It eliminates JPA boilerplate with the Active Record or Repository pattern.

## Why Panache Over Raw JPA

Standard JPA requires entity managers, typed queries, and transaction management for every operation. Panache reduces this to single method calls.

```java
// Raw JPA
public User findByName(String name) {
    TypedQuery<User> query = em.createQuery(
        "SELECT u FROM User u WHERE u.name = :name", User.class);
    query.setParameter("name", name);
    return query.getSingleResult();
}

// Panache Active Record
public static User findByName(String name) {
    return find("name", name).firstResult();
}
```

## Step by Step

### 1. Add Extensions

```bash
mvn quarkus:add-extension -Dextensions="jdbc-postgresql,hibernate-orm-panache"
```

### 2. Define an Entity (Active Record Pattern)

```java
@Entity
@Table(name = "products")
public class Product extends PanacheEntity {

    @Column(nullable = false)
    public String name;

    @Column(nullable = false)
    public BigDecimal price;

    @Column
    public String category;

    @Column(name = "created_at")
    public LocalDateTime createdAt = LocalDateTime.now();
}
```

`PanacheEntity` provides `id` field and all CRUD methods. Fields are public -- Panache generates getters/setters via bytecode enhancement.

### 3. Basic CRUD Operations

```java
@Path("/products")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ProductResource {

    @GET
    public List<Product> list() {
        return Product.listAll();
    }

    @GET
    @Path("/{id}")
    public Product get(@PathParam("id") Long id) {
        Product product = Product.findById(id);
        if (product == null) {
            throw new NotFoundException();
        }
        return product;
    }

    @POST
    @Transactional
    @Status(Response.Status.CREATED)
    public Product create(Product product) {
        product.persist();
        return product;
    }

    @PUT
    @Path("/{id}")
    @Transactional
    public Product update(@PathParam("id") Long id, Product updated) {
        Product product = Product.findById(id);
        if (product == null) {
            throw new NotFoundException();
        }
        product.name = updated.name;
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

### 4. Queries

```java
// Find by field
Product.find("category", "electronics").list();

// Named parameters
Product.find("name LIKE :name AND price < :max",
    Parameters.with("name", "%laptop%").and("max", 1000))
    .list();

// Count
long count = Product.count("category", "electronics");

// Delete
Product.delete("category", "deprecated");

// Sorted, paginated
Product.findAll(Sort.by("price").descending()).page(Page.of(0, 20)).list();
```

### 5. Repository Pattern Alternative

If you prefer separating persistence logic from entities:

```java
@ApplicationScoped
public class ProductRepository implements PanacheRepository<Product> {

    public List<Product> findByCategory(String category) {
        return find("category", category).list();
    }

    public List<Product> findBelowPrice(BigDecimal maxPrice) {
        return find("price < ?1 ORDER BY price", maxPrice).list();
    }
}
```

Inject and use:

```java
@Inject ProductRepository repo;

@GET
@Path("/category/{cat}")
public List<Product> byCategory(@PathParam("cat") String cat) {
    return repo.findByCategory(cat);
}
```

### 6. Schema Migrations

```bash
mvn quarkus:add-extension -Dextensions="flyway"
```

```properties
# application.properties
quarkus.flyway.migrate-at-start=true
quarkus.datasource.db-kind=postgresql
```

Create migration files in `src/main/resources/db/migration/`:

```sql
-- V1.0.0__Create_products.sql
CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    name        VARCHAR(255) NOT NULL,
    price       DECIMAL(10,2) NOT NULL,
    category    VARCHAR(100),
    created_at  TIMESTAMP DEFAULT NOW()
);
```

### 7. Dev Services (No Setup Required)

With the PostgreSQL extension added, `mvn quarkus:dev` starts a PostgreSQL container automatically. No configuration needed for development.

```properties
# application.properties -- nothing needed for dev mode
# Dev Services provisions PostgreSQL automatically
quarkus.hibernate-orm.database.generation=drop-and-create
```

Previous: [REST API](02-rest-api.md) | Next: [Reactive Programming](04-reactive.md)
