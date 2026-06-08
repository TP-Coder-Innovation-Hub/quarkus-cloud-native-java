# Containerization

``

Quarkus builds optimized container images with minimal configuration. The resulting images are small, fast to start, and use minimal resources.

## Step by Step

### 1. JVM Mode Container

```bash
# Build the container image
mvn package \
    -Dquarkus.container-image.build=true \
    -Dquarkus.container-image.name=my-service

# Run it
docker run -p 8080:8080 my-service:latest
```

### 2. Native Mode Container

```bash
# Build native image inside a container (no local GraalVM needed)
mvn package -Dnative \
    -Dquarkus.native.container-build=true \
    -Dquarkus.container-image.build=true

# Run it
docker run -p 8080:8080 my-service:latest
```

The native container uses a distroless base image. Typical size: 50--100 MB.

### 3. Configure in application.properties

```properties
# Container image settings
quarkus.container-image.group=my-org
quarkus.container-image.name=my-service
quarkus.container-image.tag=1.0.0
quarkus.container-image.registry=ghcr.io

# Push after build
quarkus.container-image.push=true

# Docker or Podman
quarkus.docker.docker-executable-name=docker

# JVM image base
quarkus.docker.base-image=registry.access.redhat.com/ubi8/openjdk-21:1.20

# Native image base (minimal)
quarkus.docker.base-image-native=quay.io/quarkus/quarkus-micro-image:2.0
```

### 4. Dockerfile Customization

For more control, create `src/main/docker/Dockerfile.native`:

```dockerfile
FROM quay.io/quarkus/quarkus-micro-image:2.0
WORKDIR /work/
COPY target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
USER 1001
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

Build and run:

```bash
# Build native executable
mvn package -Dnative -Dquarkus.native.container-build=true

# Build image
docker build -f src/main/docker/Dockerfile.native -t my-service:1.0.0 .

# Run
docker run -p 8080:8080 my-service:1.0.0
```

### 5. Kubernetes Deployment

```bash
# Generate Kubernetes manifests
mvn package \
    -Dquarkus.kubernetes.generate=true \
    -Dquarkus.container-image.build=true

# Apply
kubectl apply -f target/kubernetes/kubernetes.yml
```

Or deploy directly:

```bash
# Deploy to current Kubernetes context
mvn package -Dquarkus.kubernetes.deploy=true
```

Generated manifests include resource limits optimized for Quarkus native:

```yaml
resources:
  requests:
    memory: "32Mi"
    cpu: "100m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

### 6. Multi-Stage Build (CI Pipeline)

```dockerfile
# Stage 1: Build
FROM quay.io/quarkus/ubi-quarkus-mandrel-builder-image:jdk-21 AS build
COPY pom.xml /code/
COPY src /code/src
WORKDIR /code
RUN mvn package -Dnative -DskipTests

# Stage 2: Runtime
FROM quay.io/quarkus/quarkus-micro-image:2.0
COPY --from=build /code/target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
USER 1001
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

## Image Size Comparison

| Base Image | Mode | Typical Size |
|------------|------|-------------|
| JDK 21 (Ubuntu) | JVM | 400--500 MB |
| JDK 21 (Distroless) | JVM | 200--300 MB |
| Distroless / Micro | Native | 50--100 MB |
| Scratch | Native | 30--60 MB |

Previous: [GraalVM Native](01-graalvm-native.md) | Next: [Microservice Patterns](03-microservice-patterns.md)
