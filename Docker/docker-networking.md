# Docker Networking

## 1. What Is Docker Networking?

**Definition:** Docker networking is a mechanism that allows Docker containers to communicate with each other, the host machine, and external networks.

**Real-world example:** Imagine an application running in one container and a MySQL database running in another container. Docker networking allows the application to communicate with the database.

## 2. Types of Docker Networks

### A. Bridge Network

**Definition:** A bridge network allows containers on the same Docker host to communicate with each other.

**Real-world example:** Samaj, ek application container aahe ani ek MySQL database container aahe. Donhi same bridge network var astil, tar application database la container name vaprun connect karu shakto.

**Command:**

```bash
docker network create my-network

docker run -d --name app1 --network my-network nginx

docker run -d --name app2 --network my-network nginx
```

**Important point:** User-defined bridge networks support communication by container name through Docker's built-in DNS.

### B. Host Network

**Definition:** A host network allows a container to share the host machine's network namespace.

**Real-world example:** Container host machine cha network directly use karto, rather than having its own separate network namespace.

**Command (Linux):**

```bash
docker run --network host nginx
```

**Important point:** Host networking reduces network isolation between the container and host.

### C. None Network

**Definition:** A none network disables normal external network connectivity for a container.

**Real-world example:** Ek isolated task aahe jyala internet kiwa other containers sobat network communication karaychi garaj nahi.

**Command:**

```bash
docker run --network none nginx
```

**Important point:** The loopback interface remains available, but normal external network connectivity is disabled.

### D. Overlay Network

**Definition:** An overlay network allows containers running on different Docker hosts to communicate with each other.

**Real-world example:** Company che application don vegveglya servers var run hot aahe. Docker Swarm madhye overlay network tyanna communicate karayla madat karto.

**Command:**

```bash
docker network create \
  --driver overlay \
  my-overlay
```

**Important point:** Overlay networking is commonly used in Docker Swarm for communication across multiple hosts.

### E. Macvlan Network

**Definition:** A macvlan network assigns containers their own MAC addresses, allowing them to appear as separate devices on a physical network.

**Real-world example:** Company chya existing physical network var container la separate device sarkha dakhvaycha asel, tar Macvlan use karu shakto.

**Important point:** Macvlan requires suitable network-interface configuration and support from the surrounding network.

## 3. Important Docker Network Commands

| Command | Purpose |
|---|---|
| `docker network ls` | Lists Docker networks |
| `docker network create my-network` | Creates a bridge network by default |
| `docker network inspect my-network` | Shows network details |
| `docker network connect my-network app1` | Connects an existing container to a network |
| `docker network disconnect my-network app1` | Disconnects a container from a network |
| `docker network rm my-network` | Removes a network if no containers are attached |

## 4. Interview Question

**Q: Which Docker network is commonly used for standalone containers?**

**Answer:** A bridge network is commonly used for standalone containers running on the same Docker host. A user-defined bridge network allows containers to communicate using container names and provides network isolation from containers on other networks.

**Q: What is the difference between bridge and overlay networks?**

**Answer:** A bridge network connects containers on a single Docker host, whereas an overlay network enables communication across multiple Docker hosts.

## 5. Quick Revision

- **Bridge:** Communication between containers on the same Docker host.
- **Host:** Container shares the host's network namespace.
- **None:** No normal external network connectivity.
- **Overlay:** Communication across multiple Docker hosts.
- **Macvlan:** Containers appear as separate devices on a physical network.

## 6. Practice Commands

```bash
docker network ls
docker network create practice-network
docker network inspect practice-network
docker network rm practice-network
```

**Note:** Remove the practice network only after confirming that no containers are attached to it.
