# Docker Fundamentals — Complete Notes

## 1. What Is Docker?

**Definition:** Docker is a platform used to build, package, distribute, and run applications in containers.

**Simple Explanation:**

Docker packages an application and its required dependencies into an image. We can use that image to create containers that run the application in a consistent environment.

**Real-World Example:**

Bhau, samaj tu Spring Boot application develop keli aahe. Tujhya laptop var ti perfectly run hote, pan server var Java version kiwa dependencies different aslyamule error yeto.

Docker helps package the application and its required runtime dependencies so it behaves more consistently across different environments.

### Important Docker Components

- **Dockerfile:** Contains instructions to build an image.
- **Docker Image:** A packaged template containing the application and its dependencies.
- **Docker Container:** An instance of a Docker image.
- **Docker Engine:** Builds and runs containers.
- **Docker Compose:** Configures and runs application services together.

### Interview Answer

"Docker is a containerization platform that allows us to package applications and their dependencies into images and run them in isolated containers. It helps improve consistency and portability across development, testing, and production environments."

---

## 2. Why Do We Use Docker?

Docker helps solve common application deployment problems.

### A. Environment Consistency

An application may work on a developer's laptop but fail on a different server because of different dependencies or configurations.

Docker packages the application and its runtime dependencies into an image, helping reduce these inconsistencies.

**Example:** A Spring Boot application requires a particular Java runtime. We can package it with a compatible Java runtime image.

### B. Portability

Docker images can be distributed through container registries and run on compatible Docker environments.

**Example:** Bhau, laptop var image build keli ani ti compatible cloud server var deploy keli.

### C. Faster Deployment

Containers can generally start faster than full virtual machines because they do not require a separate guest operating system.

### D. Efficient Resource Utilization

Containers share the host kernel instead of each running a separate guest operating system.

### E. Scalability

We can run multiple instances of an application and use orchestration tools to manage scaling and deployment.

### Interview Answer

"We use Docker to package applications with their dependencies, maintain consistent environments, simplify deployments, improve resource utilization, and support scalable application architectures."

---

## 3. Advantages of Docker

| Advantage | Explanation |
|---|---|
| Portability | Images can run across compatible environments |
| Consistency | Reduces differences between development and production |
| Lightweight | Containers generally use fewer resources than full VMs |
| Fast startup | Containers can start quickly |
| Scalability | Multiple container instances can run when needed |
| Isolation | Applications have isolated process and filesystem environments |
| Versioning | Images can be tagged and distributed by version |
| CI/CD integration | Useful for automated build, test, and deployment pipelines |
| Ecosystem | Provides tools for networking, storage, registries, and orchestration |

**Real-world example:** If your company has ten services, you can package each service in an image instead of manually installing and configuring every dependency on every server.

## 4. Disadvantages of Docker

| Disadvantage | Explanation |
|---|---|
| Learning curve | Requires understanding images, containers, networks, volumes, and configuration |
| Persistent storage | Data requires appropriate volumes, mounts, and backup strategies |
| Security risks | Misconfigured containers, excessive privileges, or vulnerable images can create risks |
| Networking complexity | Communication between services and external systems requires proper configuration |
| Monitoring complexity | Many containers require suitable centralized logging and monitoring |
| Kernel dependency | Containers share the host kernel, so they are not equivalent to full OS virtualization |
| Production management | Large deployments may need orchestration and additional operational expertise |

**Important:** Docker is not automatically secure or lightweight in every situation. Good configuration, image maintenance, resource limits, and monitoring are important.

### Interview Answer

"Although Docker offers portability and efficient resource utilization, it also introduces challenges such as persistent storage management, networking complexity, security configuration, and monitoring across multiple containers."

---

## 5. Virtual Machines vs Docker Containers

### What Is a Virtual Machine?

**Definition:** A virtual machine is a software-based computer that runs its own operating system on virtualized hardware.

A hypervisor creates and manages virtual machines.

### What Is a Docker Container?

**Definition:** A Docker container is an isolated running instance of an image that shares the host operating system's kernel.

### Architecture Comparison

**Virtual Machines:**

```text
+------------------------------+
|         Applications         |
+------------------------------+
|      Guest Operating System  |
+------------------------------+
|         Hypervisor           |
+------------------------------+
|        Host Hardware         |
+------------------------------+
```

**Docker Containers:**

```text
+------------------------------+
| App A | App B | App C        |
+------------------------------+
| Container Runtime / Isolation|
+------------------------------+
|       Host Operating System  |
+------------------------------+
|        Host Hardware         |
+------------------------------+
```

A container engine such as Docker Engine manages containers. Containers do not each need their own guest operating system.

### Main Differences

| Feature | Virtual Machine | Docker Container |
|---|---|---|
| Virtualization | Virtualizes hardware | Uses OS-level virtualization |
| Operating system | Has its own guest OS | Shares the host kernel |
| Resource usage | Generally higher | Generally lower |
| Startup time | Usually slower | Usually faster |
| Isolation | Strong isolation boundary through virtualization | Process-level isolation; configuration matters |
| Portability | VM images include a guest OS | Container images package applications and dependencies |
| Typical use | Different OS environments, strong isolation requirements | Microservices, application packaging, CI/CD |

**Real-world example:**

Bhau, VM mhanje ekach building madhye separate full flats, pratyek flat la tyachi independent setup aahe.

Containers mhanje ekach building madhye separate rooms, shared building infrastructure vaprun chalnare. Donhi isolated aahet, pan isolation cha level ani resource requirements vegle aahet.

### Can Docker Run Inside a VM?

Yes. Docker Engine can run inside a Linux virtual machine. This is common in cloud environments.

For example:

```text
Cloud Server
    |
    v
Linux Virtual Machine
    |
    v
Docker Engine
    |
    +---- Container 1
    |
    +---- Container 2
    |
    +---- Container 3
```

### Interview Answer

"A virtual machine runs its own guest operating system on virtualized hardware, whereas a Docker container shares the host kernel and provides process-level isolation. Containers are generally more lightweight, while VMs provide a separate guest operating system and a different isolation boundary."

---

## 6. Docker Architecture

**Definition:** Docker architecture is the client-server architecture used by Docker to build images, manage containers, and communicate with registries.

Docker's main components include the Docker client, Docker daemon, Docker Engine APIs, images, containers, networks, volumes, and registries.

### Architecture Diagram

```text
          Docker Client
       (docker commands)
                |
                | Docker API
                v
       +-------------------+
       |   Docker Daemon   |
       |     dockerd       |
       +-------------------+
          |      |      |
          v      v      v
       Images  Containers Networks
          |
          v
       Registries

       Volumes provide
       persistent storage
       for containers
```

### A. Docker Client

**Definition:** The Docker client is the interface used to send commands to Docker.

Examples:

```bash
docker build -t my-app .
docker run my-app
docker ps
```

**Explanation:** When you execute a Docker command, the client communicates with the Docker daemon through the Docker API.

### B. Docker Daemon (`dockerd`)

**Definition:** The Docker daemon is a background service responsible for managing Docker objects and responding to Docker API requests.

It manages tasks such as building images, creating containers, networks, and volumes.

### C. Docker Engine API

**Definition:** The Docker Engine API provides an interface through which clients can communicate with the Docker daemon.

The Docker CLI commonly uses this API to request operations.

### D. Docker Images

**Definition:** A Docker image is a read-only template used to create containers.

Images can be built locally or pulled from a registry.

### E. Docker Containers

**Definition:** A container is an isolated instance of an image with a writable container layer and its runtime configuration.

Containers can be started, stopped, and removed.

### F. Docker Registry

**Definition:** A Docker registry stores and distributes Docker images.

Examples include Docker Hub and private container registries.

Common commands:

```bash
docker pull nginx:alpine
docker push myusername/my-app:1.0
```

Pushing an image requires the appropriate registry name, authentication, and permissions.

### G. Docker Networks

**Definition:** Docker networks provide connectivity between containers and other network endpoints.

Example:

```bash
docker network create my-network
```

### H. Docker Volumes

**Definition:** Docker volumes provide persistent storage that can exist independently of a container's lifecycle.

Example:

```bash
docker volume create my-volume
```

### How Docker Architecture Works

Suppose you want to run Nginx.

**Step 1:** The user runs:

```bash
docker run nginx:alpine
```

**Step 2:** The Docker client sends a request to the daemon.

**Step 3:** The daemon checks whether the image is available locally.

**Step 4:** If the image is missing, Docker can pull it from a registry.

**Step 5:** The daemon creates and starts the container.

**Step 6:** The container runs the Nginx process.

**Interview Answer:**

"Docker follows a client-server architecture. The Docker client communicates with the Docker daemon through the Docker API. The daemon manages images, containers, networks, and volumes, while registries store and distribute images."

---

## 7. Types of Deployment Strategies

In DevOps, deployment strategies describe how a new application version is released and how traffic moves between versions.

These strategies are not exclusive to Docker. They can be implemented using Docker, virtual machines, Kubernetes, or other deployment platforms.

### A. Recreate Deployment

**Definition:** The existing application instances are stopped before the new version is deployed.

**How it works:**

```text
Old Version Running
        |
        v
Old Version Stopped
        |
        v
New Version Started
```

**Real-world example:** Bhau, shop band karun junya setup cha replacement karto ani nantar navin setup start karto.

**Advantages:**
- Simple to implement.
- Easy to understand.

**Disadvantages:**
- Can cause downtime during deployment.

**Use case:** Development environments or applications where brief downtime is acceptable.

### B. Rolling Deployment

**Definition:** The new version is deployed gradually, replacing old instances in batches.

**How it works:**

```text
Old Old Old Old
      |
      v
New Old Old Old
      |
      v
New New Old Old
      |
      v
New New New New
```

**Real-world example:** Bhau, hotel madhle sagle workers ekach veli replace na karta thodya-thodya batches madhye replace karto, so service chalu rahte.

**Advantages:**
- Can reduce downtime.
- Gradual replacement of instances.

**Disadvantages:**
- Old and new versions may run simultaneously.
- Requires compatibility between versions and careful health checks.

**Use case:** Web applications running multiple instances behind a load balancer.

### C. Blue-Green Deployment

**Definition:** Two environments are maintained: one runs the current version, while the other runs the new version. Traffic is switched to the new environment after validation.

**How it works:**

```text
Blue Environment
(Current Version)
        |
        v
Test Green Environment
(New Version)
        |
        v
Switch Traffic to Green
```

**Real-world example:** Bhau, ek shop chalu thevto ani dusri identical shop ready karto. Navin shop test kelyavar customers la tikde shift karto.

**Advantages:**
- Allows testing before switching traffic.
- Can enable fast rollback by switching traffic back, if the old environment remains healthy.

**Disadvantages:**
- Requires additional infrastructure.
- Database changes and shared state can complicate rollback.

**Use case:** Applications that need a controlled release and quick rollback option.

### D. Canary Deployment

**Definition:** A new application version is released to a small subset of users or traffic before being rolled out more widely.

**How it works:**

```text
100% Traffic
     |
     v
Small Percentage to New Version
     |
     v
Monitor Errors and Performance
     |
     v
Increase Traffic Gradually
```

**Real-world example:** Bhau, navin feature aadhi fakt kahi customers la deto. Feedback changla asel tar baki customers sathi pan release karto.

**Advantages:**
- Limits initial exposure to the new version.
- Helps detect problems before a full rollout.

**Disadvantages:**
- Requires traffic routing and monitoring.
- Adds operational complexity.

**Use case:** High-traffic applications where a gradual release reduces deployment risk.

### E. A/B Deployment or Testing

**Definition:** Different application variants are shown to different user groups to compare outcomes.

**Real-world example:** E-commerce website var kahi users na button design A ani dusrya users na design B dakhavto, mag conversion rates compare karto.

**Important distinction:** A/B testing is primarily an experimentation technique. It may use deployment and traffic-routing mechanisms, but it is not the same as a standard infrastructure rollout strategy.

### Deployment Strategy Comparison

| Strategy | How it works | Main benefit | Main drawback |
|---|---|---|---|
| Recreate | Replace the old version all at once | Simplicity | Downtime may occur |
| Rolling | Replace instances gradually | Gradual rollout | Mixed versions may coexist |
| Blue-Green | Switch traffic between two environments | Controlled cutover and potential quick rollback | Extra infrastructure |
| Canary | Send a small portion of traffic to the new version | Limits initial risk | More monitoring and routing complexity |
| A/B Testing | Compare variants across user groups | Measures user outcomes | Requires experiment design and analysis |

### Interview Answer

"Common deployment strategies include Recreate, Rolling, Blue-Green, and Canary deployments. Recreate replaces the old version, Rolling updates instances gradually, Blue-Green switches traffic between two environments, and Canary gradually exposes the new version to a small portion of users."

---

## 8. Important Interview Questions

### Q1. What is Docker?

Docker is a platform used to package and run applications in containers.

### Q2. Why do we use Docker?

We use Docker to improve environment consistency, portability, deployment efficiency, and resource utilization.

### Q3. What is the difference between a VM and a container?

A VM runs its own guest OS, whereas a container shares the host kernel and provides process-level isolation.

### Q4. Explain Docker architecture.

Docker uses a client-server architecture. The client communicates with the daemon through the Docker API, and the daemon manages images, containers, networks, and volumes.

### Q5. What is a Docker registry?

A Docker registry stores and distributes Docker images.

### Q6. What is a deployment strategy?

A deployment strategy defines how a new application version is released and how traffic moves between versions.

### Q7. What is the difference between Blue-Green and Canary deployment?

Blue-Green uses two environments and switches traffic between them, while Canary gradually exposes a new version to a small percentage of traffic or users.

### Q8. Does Docker replace virtual machines?

No. Docker containers and VMs solve different problems and can be used together. Docker can run inside a VM.

---

## 9. Quick Revision

- **Docker:** Platform for packaging and running applications in containers.
- **Dockerfile:** Instructions for building an image.
- **Image:** Packaged template for containers.
- **Container:** Isolated running instance of an image.
- **VM:** Virtual machine with its own guest OS.
- **Docker architecture:** Client, daemon, API, images, containers, networks, volumes, and registries.
- **Recreate:** Stop old instances and start the new version.
- **Rolling:** Replace instances gradually.
- **Blue-Green:** Switch traffic between two environments.
- **Canary:** Release to a small portion of traffic first.
- **A/B testing:** Compare application variants using user outcomes.

## Final Interview Summary

"Docker is a containerization platform that helps package applications and their dependencies into portable images. Docker containers generally use fewer resources than virtual machines because they share the host kernel. Docker uses a client-server architecture in which the client communicates with the daemon through the Docker API. Common deployment strategies include Recreate, Rolling, Blue-Green, and Canary deployments, each offering different trade-offs in availability, complexity, and deployment risk."
