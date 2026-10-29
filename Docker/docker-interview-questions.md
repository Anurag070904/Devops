# Docker Interview Questions and Answers
## Beginner to Intermediate — DevOps Interview Preparation

## Section 1: Docker Fundamentals

### Q1. What is Docker?

**Answer:**

Docker is a containerization platform that allows us to build, package, distribute, and run applications in containers.

It packages an application with its required dependencies into an image, helping the application run consistently across different environments.

### Q2. Why do we use Docker?

**Answer:**

We use Docker to:

- Maintain consistency across development, testing, and production environments.
- Package applications with their dependencies.
- Simplify application deployment.
- Improve resource utilization.
- Support scalable applications and CI/CD pipelines.

**Real-world example:** An application works on a developer's laptop but fails on a server because of different dependencies. Docker helps reduce these environment-related problems.

### Q3. What is containerization?

**Answer:**

Containerization is a technology that packages an application and its dependencies into an isolated environment called a container.

Containers share the host operating system's kernel instead of each requiring a separate guest operating system.

### Q4. What is the difference between a Docker image and a Docker container?

**Answer:**

A Docker image is a read-only template used to create containers. A container is an instance of an image with its own runtime configuration and writable layer.

| Docker Image | Docker Container |
|---|---|
| Template for creating containers | Instance created from an image |
| Mostly read-only | Has a writable container layer |
| Can create multiple containers | Can be started, stopped, and removed |

### Q5. What is the difference between Docker and a virtual machine?

**Answer:**

A virtual machine runs its own guest operating system on virtualized hardware, whereas a Docker container shares the host kernel and provides process-level isolation.

Containers are generally more lightweight and start faster, while VMs provide a separate guest OS and a different isolation boundary.

### Q6. What are the advantages and disadvantages of Docker?

**Answer:**

**Advantages:**
- Portability and consistency.
- Efficient resource utilization.
- Faster startup.
- Easier application packaging.
- Integration with CI/CD workflows.

**Disadvantages:**
- Networking and storage can require additional configuration.
- Security depends on proper configuration.
- Large container deployments require monitoring and orchestration.
- Containers share the host kernel, unlike traditional VMs with separate guest OSs.

### Q7. What is Docker Engine?

**Answer:**

Docker Engine is the technology used to build and run containers. It includes the Docker daemon, APIs, and Docker CLI components.

---

## Section 2: Docker Architecture

### Q8. Explain Docker architecture.

**Answer:**

Docker follows a client-server architecture. The Docker client communicates with the Docker daemon through the Docker API. The daemon manages images, containers, networks, and volumes.

The main components are:

1. **Docker Client:** Sends Docker commands.
2. **Docker Daemon:** Manages Docker objects and operations.
3. **Docker Engine API:** Provides communication between clients and the daemon.
4. **Docker Images:** Templates used to create containers.
5. **Docker Containers:** Isolated instances of images.
6. **Docker Registry:** Stores and distributes images.

### Q9. What is a Docker daemon?

**Answer:**

The Docker daemon, `dockerd`, is a background service responsible for managing Docker objects such as images, containers, networks, and volumes.

### Q10. What is a Docker registry?

**Answer:**

A Docker registry is a service that stores and distributes Docker images.

Examples include Docker Hub and private container registries.

**Commands:**

```bash
docker pull nginx:alpine
docker push username/my-app:1.0
```

The push command requires the correct registry location, authentication, and permissions.

### Q11. What happens when you execute `docker run nginx`?

**Answer:**

1. Docker receives the request through the CLI.
2. The Docker daemon checks whether the image is available locally.
3. If the image is missing, Docker attempts to pull it from a registry.
4. Docker creates a container.
5. Docker starts the container and its configured process.

The container runs Nginx, but its web port is not automatically published to the host.

### Q12. What is the difference between `docker create`, `docker start`, and `docker run`?

**Answer:**

- `docker create` creates a container without starting it.
- `docker start` starts an existing stopped container.
- `docker run` creates and starts a new container.

**Example:**

```bash
docker create --name test nginx:alpine
docker start test
```

---

## Section 3: Docker Commands

### Q13. How do you list running containers?

**Answer:**

```bash
docker ps
```

To list all containers, including stopped ones:

```bash
docker ps -a
```

### Q14. How do you list Docker images?

**Answer:**

```bash
docker images
```

This displays locally available images.

### Q15. How do you stop and remove a container?

**Answer:**

```bash
docker stop my-container
docker rm my-container
```

The first command stops the container. The second removes it.

### Q16. How do you remove a Docker image?

**Answer:**

```bash
docker rmi nginx:alpine
```

The image must not be in use by containers in a way that prevents removal.

### Q17. How do you access a running container's terminal?

**Answer:**

```bash
docker exec -it my-container sh
```

If Bash is installed, you can use:

```bash
docker exec -it my-container bash
```

`docker exec` runs a command inside an already running container.

### Q18. How do you check container logs?

**Answer:**

```bash
docker logs my-container
```

To follow new log output:

```bash
docker logs -f my-container
```

This is useful when troubleshooting an application that fails to start or behaves unexpectedly.

---

## Section 4: Dockerfile

### Q19. What is a Dockerfile?

**Answer:**

A Dockerfile is a text file containing instructions used to build a Docker image.

Common instructions include `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`, and `ENTRYPOINT`.

### Q20. What is the difference between RUN, CMD, and ENTRYPOINT?

**Answer:**

- **RUN:** Executes commands during image building.
- **CMD:** Specifies the default command or arguments when a container starts.
- **ENTRYPOINT:** Defines the main executable for a container.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

RUN pip install --no-cache-dir flask

CMD ["python", "app.py"]
```

Here, `RUN` installs Flask while building the image, and `CMD` defines the default runtime command.

**Interview shortcut:** RUN = build time; CMD and ENTRYPOINT = container startup.

### Q21. What is the difference between COPY and ADD?

**Answer:**

`COPY` copies files and directories from the build context into the image.

`ADD` supports additional features, such as extracting local tar archives.

For ordinary file copying, `COPY` is generally preferred because its behavior is more straightforward.

### Q22. What is the purpose of EXPOSE?

**Answer:**

`EXPOSE` documents the port on which an application is intended to listen inside the container.

Example:

```dockerfile
EXPOSE 8080
```

It does not publish the port to the host. For that, we can use:

```bash
docker run -p 8080:8080 my-app
```

### Q23. What is a multi-stage Docker build?

**Answer:**

A multi-stage build uses multiple `FROM` instructions in a Dockerfile to separate build dependencies from the final runtime image.

It can reduce image size and avoid including unnecessary build tools in the final image.

Example for a Java application:

```dockerfile
# Stage 1: Build the application
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn package -DskipTests

# Stage 2: Run the application
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]
```

This example assumes the Maven project builds a runnable JAR under `target/`.

### Q24. What is a .dockerignore file?

**Answer:**

A `.dockerignore` file specifies files and directories that should be excluded from the Docker build context.

Example:

```text
.git
.env
node_modules
__pycache__
```

It can reduce build-context size and help prevent unnecessary or sensitive files from being copied into an image.

### Q25. What is the difference between ARG and ENV?

**Answer:**

- `ARG` defines a variable available during image building.
- `ENV` sets an environment variable in the image that is available to containers created from it, unless overridden.

Neither should be treated as a secure way to store secrets.

---

## Section 5: Docker Networking

### Q26. What is Docker networking?

**Answer:**

Docker networking allows containers to communicate with each other, the host machine, and external networks.

### Q27. What are the main types of Docker networks?

**Answer:**

1. **Bridge:** Connects containers on the same Docker host.
2. **Host:** Allows a container to share the host's network namespace.
3. **None:** Disables normal external network connectivity.
4. **Overlay:** Connects containers across multiple Docker hosts, commonly in Docker Swarm.
5. **Macvlan:** Allows containers to appear as separate devices on a physical network.

### Q28. What is the difference between bridge and host networking?

**Answer:**

Bridge networking gives containers a separate network namespace and connects them through a virtual bridge.

Host networking shares the host's network namespace, reducing network isolation between the container and host.

### Q29. How can two containers communicate with each other?

**Answer:**

Containers attached to the same user-defined bridge network can communicate using their container names or network aliases.

Example:

```bash
docker network create app-network

docker run -d \
  --name web \
  --network app-network \
  nginx:alpine

docker run -d \
  --name client \
  --network app-network \
  alpine sleep 3600
```

The client container can connect to `web` by name over the network, provided the relevant service is listening.

### Q30. What is port mapping in Docker?

**Answer:**

Port mapping connects a port on the host machine to a port inside a container.

Example:

```bash
docker run -d -p 8080:80 nginx:alpine
```

Here, host port `8080` maps to container port `80`.

The Nginx server can then be accessed at `http://localhost:8080` on the Docker host.

---

## Section 6: Docker Volumes and Storage

### Q31. What is a Docker volume?

**Answer:**

A Docker volume is a storage mechanism managed by Docker that allows data to persist independently of a container's lifecycle.

### Q32. What is the difference between a volume and a bind mount?

**Answer:**

A volume is managed by Docker, whereas a bind mount maps a specific file or directory from the host machine into a container.

Volumes are commonly used for database data. Bind mounts are useful for sharing source code during development.

### Q33. What happens to volume data when a container is deleted?

**Answer:**

A named volume normally remains available when its container is deleted. It can be mounted into another container.

However, explicitly deleting the volume can permanently remove its data. Volumes also require a separate backup strategy.

### Q34. How do you create and inspect a volume?

**Answer:**

```bash
docker volume create my-volume
docker volume ls
docker volume inspect my-volume
```

To remove an unused volume:

```bash
docker volume rm my-volume
```

---

## Section 7: Docker Compose

### Q35. What is Docker Compose?

**Answer:**

Docker Compose is a tool used to define and run multi-container Docker applications through a YAML configuration file, commonly named `compose.yaml`.

### Q36. What is the difference between Dockerfile and Docker Compose?

**Answer:**

A Dockerfile defines how to build an image. Docker Compose defines how application services should be configured and run together.

Compose can use existing images or build custom images using Dockerfiles.

### Q37. Is a Dockerfile compulsory when using Docker Compose?

**Answer:**

No. If we use an existing image, we can specify it directly using the `image` property.

A Dockerfile is typically used when we need to build a custom image from application code.

### Q38. How do you start and stop a Compose application?

**Answer:**

Start services in the background:

```bash
docker compose up -d
```

List services:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs
```

Stop and remove the application's containers and network:

```bash
docker compose down
```

Named volumes normally remain unless they are explicitly removed.

---

## Section 8: Image Layers, Security, and Troubleshooting

### Q39. What are Docker image layers?

**Answer:**

Docker images are composed of layers representing filesystem changes. Docker can reuse cached layers during builds, which can make repeated builds faster.

### Q40. What is Docker image caching?

**Answer:**

Docker build caching allows Docker to reuse the results of previous build steps when the relevant inputs have not changed.

For example, copying dependency files and installing dependencies before copying frequently changing source code can improve build efficiency.

### Q41. How do you troubleshoot a container that exits immediately?

**Answer:**

I would:

1. Check the container status.
2. Inspect the logs.
3. Review the startup command.
4. Check the configuration and environment variables.
5. Inspect the container details if needed.

Commands:

```bash
docker ps -a
docker logs my-container
docker inspect my-container
```

A container normally stops when its main process exits.

### Q42. How do you improve Docker container security?

**Answer:**

- Use trusted and maintained base images.
- Avoid running applications as root when practical.
- Do not embed passwords or tokens in images.
- Keep images and dependencies updated.
- Apply appropriate resource limits and security restrictions.
- Expose only the required ports.
- Use least-privilege access.

### Q43. What is the difference between CMD and ENTRYPOINT at runtime?

**Answer:**

With no `ENTRYPOINT`, a command supplied to `docker run` replaces the image's default `CMD`.

With an exec-form `ENTRYPOINT`, runtime arguments are normally passed to the entrypoint, replacing default `CMD` arguments when provided.

### Q44. What is Docker Compose depends_on?

**Answer:**

`depends_on` defines service startup ordering in Docker Compose. By itself, it does not guarantee that a dependency is ready to accept requests. Health checks and appropriate readiness handling may also be required.

---

## Section 9: Deployment and Real-World Scenarios

### Q45. What is a deployment strategy?

**Answer:**

A deployment strategy defines how a new application version is released and how traffic moves between application versions.

Common strategies include Recreate, Rolling, Blue-Green, and Canary deployments.

### Q46. What is the difference between Blue-Green and Canary deployment?

**Answer:**

Blue-Green deployment uses two environments and switches traffic to the new environment after validation.

Canary deployment initially sends a small portion of traffic to the new version, then gradually increases exposure if the application performs well.

### Q47. How would you Dockerize a Spring Boot application?

**Answer:**

I would build the Spring Boot JAR, write a Dockerfile using a compatible Java runtime image, copy the JAR into the image, define the startup command, build the image, and run the container.

For applications that also require MySQL or Redis, I could use Docker Compose to configure and run those services together.

### Q48. How would you deploy an application with a database?

**Answer:**

I would package the application into an image, use an appropriate database image, configure networking and credentials, and use persistent storage for database data.

I would also implement health checks, logging, monitoring, backups, and appropriate secret management.

### Q49. Does Docker replace Kubernetes?

**Answer:**

No. Docker provides container-related tooling, while Kubernetes is a container orchestration platform that manages deployments, scaling, service discovery, and recovery across a cluster.

Kubernetes can run container images built with Docker tools or other compatible image-building tools.

### Q50. How does Docker help in CI/CD?

**Answer:**

Docker allows a CI/CD pipeline to build an image, test it, publish it to a registry, and deploy the same image to different environments.

This helps improve consistency between testing and production.

---

## 10. Quick Revision — Most Important Questions

If you have limited time before an interview, prioritize these questions:

1. What is Docker and why do we use it?
2. What is the difference between an image and a container?
3. Explain Docker architecture.
4. What is the difference between a VM and a container?
5. Explain Dockerfile instructions.
6. What is the difference between RUN, CMD, and ENTRYPOINT?
7. What is Docker Compose?
8. What is the difference between Dockerfile and Docker Compose?
9. What is Docker networking?
10. What is the difference between bridge and host networking?
11. What is a Docker volume?
12. What is the difference between a volume and a bind mount?
13. What is a Docker registry?
14. What are image layers and build caching?
15. How do you troubleshoot a container that exits?
16. How do you improve container security?
17. What is a multi-stage Docker build?
18. What is a deployment strategy?
19. How would you Dockerize a Spring Boot application?
20. How does Docker help in CI/CD?

## Final Interview Tip

For each question, practise answering in three parts:

1. **Definition:** Explain the concept in one or two sentences.
2. **Example:** Give a practical example from an application.
3. **Command:** Demonstrate the relevant Docker command when appropriate.

For example:

"Docker volumes provide persistent storage independently of a container's lifecycle. I would use a named volume to store MySQL database data. I can create one using `docker volume create mysql-data`."

This approach helps you demonstrate conceptual understanding and practical knowledge during a DevOps interview.
