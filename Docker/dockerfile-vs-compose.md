# Dockerfile vs Docker Compose

## 1. What Is a Dockerfile?

**Definition:** A Dockerfile is a text file containing instructions used to build a Docker image.

**Purpose:** It defines how to package an application and its dependencies into an image.

**Common instructions:**
- `FROM` — Selects the base image.
- `WORKDIR` — Sets the working directory.
- `COPY` — Copies files into the image.
- `RUN` — Executes commands during image building.
- `CMD` — Specifies the default command or arguments at container startup.
- `ENTRYPOINT` — Defines the main executable for the container.
- `EXPOSE` — Documents the intended listening port.

**Real-world example:**

Bhau, samaj tu cake banavtos. Recipe madhe ingredients ani preparation steps astat. Dockerfile pan tasach aahe — it defines how to prepare the application image.

## 2. What Is Docker Compose?

**Definition:** Docker Compose is a tool used to define and run multi-container Docker applications using a YAML configuration file.

The commonly used configuration filename is `compose.yaml`.

**Purpose:** It helps configure and manage multiple services together, including their ports, networks, volumes and environment variables.

**Common Compose properties:**
- `services` — Defines application services.
- `image` — Specifies an existing image.
- `build` — Specifies how to build an image.
- `ports` — Maps host ports to container ports.
- `environment` — Sets environment variables.
- `volumes` — Configures storage mounts.
- `networks` — Configures service networking.
- `depends_on` — Defines service startup ordering.

**Real-world example:**

Samaj ek hotel aahe. Kitchen, billing counter ani storage room he vegvegale parts aahet. Docker Compose helps coordinate the services of an application in one configuration.

## 3. Main Difference

| Feature | Dockerfile | Docker Compose |
|---|---|---|
| What is it? | A file containing image-build instructions | A tool that reads a YAML configuration file |
| Main purpose | Build a Docker image | Configure and run application services |
| File name | `Dockerfile` | `compose.yaml` |
| Format | Dockerfile instructions | YAML |
| Can use existing images? | A Dockerfile usually defines a build | Yes, through the `image` property |
| Can build custom images? | Yes, it defines how | Yes, through the `build` property |
| Manage multiple services together? | No, not by itself | Yes |
| Is it compulsory? | No, if using existing images | No, individual containers can be managed with Docker commands |

## 4. Dockerfile vs Compose: How They Work Together

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | Used by Docker Compose
    v
Running Application Services
```

Docker Compose can also use existing images directly without a Dockerfile.

**Important:** Dockerfile and Docker Compose are not alternatives to each other. They solve different problems and are often used together.

## 5. Can We Run an Application Using Only a Dockerfile?

Yes. A Dockerfile can be used independently to build and run an application container.

For example, a Spring Boot backend can be packaged into a Docker image.

### Example Dockerfile for Spring Boot

```dockerfile
# Use a Java 21 runtime image
FROM eclipse-temurin:21-jre

# Set the working directory
WORKDIR /app

# Copy the built Spring Boot JAR into the image
COPY target/app.jar app.jar

# Document the application's intended port
EXPOSE 8080

# Start the Spring Boot application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build the image:

```bash
docker build -t spring-backend .
```

Run the container:

```bash
docker run -p 8080:8080 spring-backend
```

**Prerequisite:** The Spring Boot JAR must already exist at `target/app.jar`.

This example runs the backend container. If the application requires a MySQL database, that database must also be available and correctly configured.

## 6. When Do We Need Docker Compose?

Suppose an application requires:

- Spring Boot backend
- MySQL database
- Redis cache

We could run each container using separate Docker commands, or define the services together using Docker Compose.

### Example compose.yaml

```yaml
services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: example123
      MYSQL_DATABASE: myapp
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

Start the application:

```bash
docker compose up -d
```

Stop and remove the application's containers and network:

```bash
docker compose down
```

**Important notes:**
- The backend service uses a Dockerfile in `./backend` to build its image.
- The database service uses the existing MySQL image.
- The named volume retains database files when containers are removed, provided the volume is not deleted.
- The backend must be configured to connect to the database using the hostname `db` and appropriate credentials.
- `depends_on` does not by itself guarantee database readiness.
- The example password is for learning only; use proper secret management in real applications.

## 7. What About Flutter Applications?

A Flutter application may be a native mobile app or a web application.

### Flutter Android/iOS

A typical workflow is:

1. Build the Flutter application.
2. Generate an APK, AAB or iOS application bundle.
3. Distribute and install it on devices.
4. Deploy the Spring Boot backend separately.

A native Flutter mobile app does not normally need to run inside a Docker container.

### Flutter Web

Flutter web output can be served using Nginx in a Docker container. In that case, a Dockerfile can package the web assets, and Docker Compose can coordinate the web service with the backend and database.

### Example Architecture

```text
Flutter Mobile App
        |
        | HTTP / HTTPS
        v
Spring Boot Backend
        |
        | Database connection
        v
MySQL Database
```

The Flutter app can run on a user's device while the Spring Boot backend and MySQL database run in Docker containers on a server.

## 8. Dockerfile vs Docker Compose Commands

### Dockerfile workflow

```bash
docker build -t spring-backend .
docker run -p 8080:8080 spring-backend
```

### Docker Compose workflow

```bash
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

Docker Compose can build images and start services together, reducing the need to manage each service separately.

## 9. Important Interview Questions

### Q1. What is the difference between Dockerfile and Docker Compose?

**Answer:**

"A Dockerfile contains instructions to build a Docker image, whereas Docker Compose is used to configure and run application services together using a YAML file."

### Q2. Can we use Docker without Docker Compose?

**Answer:**

"Yes. We can build images and run containers using Docker commands without Docker Compose. Compose is optional and is particularly useful for multi-service applications."

### Q3. Can we use Docker Compose without a Dockerfile?

**Answer:**

"Yes. Docker Compose can use existing images through the `image` property. A Dockerfile is typically needed when we want to build a custom image from our application code."

### Q4. Does every service need a Dockerfile?

**Answer:**

"No. A service can use an existing image. For example, a MySQL service can use the official MySQL image directly."

### Q5. Why use Docker Compose for Spring Boot and MySQL?

**Answer:**

"Docker Compose allows us to define and manage the Spring Boot backend and MySQL database in one configuration file. It simplifies service startup, networking, port configuration and persistent storage."

### Q6. Is Docker Compose compulsory for production?

**Answer:**

"No. Docker Compose is optional. It can be useful for development, testing and suitable production deployments, while other orchestration platforms may be used depending on the application's requirements."

## 10. Quick Revision

- **Dockerfile:** Defines how to build an image.
- **Docker Image:** Packages the application and its dependencies.
- **Docker Container:** An instance of an image.
- **Docker Compose:** Configures and manages application services together.
- **`image:`** Uses an existing image.
- **`build:`** Builds a custom image from a build context.
- **Flutter mobile:** Usually built and distributed separately.
- **Spring Boot + MySQL:** Can be run together using Docker Compose.

## Final Interview Answer

"A Dockerfile defines the instructions needed to build a Docker image, while Docker Compose is a tool used to configure and run multiple application services through a YAML file. We can run an individual container using Docker commands alone, but Compose simplifies the management of applications that contain multiple services, such as a Spring Boot backend and a MySQL database."
