# Dockerfile — Complete Notes

## 1. What Is a Dockerfile?

**Definition:** A Dockerfile is a text file containing instructions used to build a Docker image.

**Simple Explanation:**

A Dockerfile tells Docker what base image to use, which files to copy, which dependencies to install, and what command to execute when a container starts.

**Real-World Example:**

Bhau, samaj tu chai banavtos. Chai banvaychi recipe madhe milk, sugar, tea powder ani preparation steps astat. Tasa Dockerfile madhe application build karaychya instructions astat.

## 2. Dockerfile, Docker Image, and Docker Container

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | docker run
    v
Docker Container
```

- **Dockerfile:** Contains the build instructions.
- **Docker Image:** Contains the packaged application and its required files and dependencies.
- **Docker Container:** An instance of an image that can run an application.

**Interview Answer:**

"A Dockerfile contains instructions to build a Docker image. A Docker image packages the application and its dependencies, and a container is an instance created from that image."

## 3. Important Dockerfile Instructions

### FROM

**Definition:** Specifies the base image used to build the Docker image.

```dockerfile
FROM python:3.12-slim
```

**Example:** A Python application can use an image that already contains Python.

### WORKDIR

**Definition:** Sets the working directory for subsequent Dockerfile instructions.

```dockerfile
WORKDIR /app
```

**Example:** Sets `/app` as the working directory inside the image.

### COPY

**Definition:** Copies files or directories from the build context into the image.

```dockerfile
COPY app.py .
```

**Example:** Copies `app.py` into the current working directory inside the image.

### RUN

**Definition:** Executes commands during the image build process.

```dockerfile
RUN pip install flask
```

**Example:** Installs Flask while building the image.

### CMD

**Definition:** Specifies the default command or arguments to execute when a container starts.

```dockerfile
CMD ["python", "app.py"]
```

**Example:** Runs the Python application when the container starts, unless overridden.

### ENTRYPOINT

**Definition:** Configures the main executable that runs when a container starts.

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

**Example:** Makes the Python application the main executable of the container.

### EXPOSE

**Definition:** Documents the port on which the application is intended to listen.

```dockerfile
EXPOSE 80
```

**Important:** `EXPOSE` does not publish the port. Use `docker run -p` to publish a port.

### ENV

**Definition:** Sets environment variables in the image.

```dockerfile
ENV APP_ENV=production
```

### ARG

**Definition:** Defines a variable that can be passed during image building.

```dockerfile
ARG APP_VERSION=1.0
```

### ADD

**Definition:** Copies files into the image and supports additional functionality, such as extracting local tar archives.

```dockerfile
ADD archive.tar /app/
```

Use `COPY` for ordinary file copying unless you specifically need an `ADD` feature.

### USER

**Definition:** Sets the user for subsequent build instructions and the default runtime user.

```dockerfile
USER appuser
```

The user must exist in the image.

### VOLUME

**Definition:** Declares a mount point intended for external or persistent storage.

```dockerfile
VOLUME ["/data"]
```

This declaration alone does not guarantee that the data will be backed up or persisted in a particular named volume.

---

## 4. Dockerfile RUN vs CMD vs ENTRYPOINT

These three instructions are especially important for DevOps interviews.

| Feature | RUN | CMD | ENTRYPOINT |
|---|---|---|---|
| Execution time | Image build time | Container startup | Container startup |
| Main purpose | Prepare the image | Set the default command or arguments | Define the main executable |
| Typical use | Install dependencies | Start an application by default | Define the application or service to run |
| Can runtime arguments affect it? | No, not directly | Yes, commonly overridden | Runtime arguments are normally passed to it |

### A. RUN

**Interview Definition:**

"RUN executes commands during image building, such as installing packages or creating files."

Example:

```dockerfile
FROM ubuntu:24.04

# Update package information during the build
RUN apt-get update

# Install curl into the image
RUN apt-get install -y curl
```

Build the image:

```bash
docker build -t my-ubuntu .
```

**Real-world example:** Bhau, ghar tayar kartana wiring, painting ani tiles lavne he preparation work aahe. Similarly, `RUN` prepares the image.

### B. CMD

**Interview Definition:**

"CMD specifies the default command or arguments for a container. They can be overridden when the container is started."

Example:

```dockerfile
FROM ubuntu:24.04

# Display a message when the container starts
CMD ["echo", "Hello Bhau!"]
```

Build and run:

```bash
docker build -t my-cmd .
docker run my-cmd
```

Output:

```text
Hello Bhau!
```

Override the default command:

```bash
docker run my-cmd echo "Hello Docker!"
```

Output:

```text
Hello Docker!
```

**Real-world example:** Tu ghari aalyavar default TV baghtos, pan aaj music aikaycha tharavlas tar default action change kartos. `CMD` is the default behavior that can be overridden.

### C. ENTRYPOINT

**Interview Definition:**

"ENTRYPOINT defines the main executable that runs when a container starts. Runtime arguments can be passed to it."

Example:

```dockerfile
FROM ubuntu:24.04

# Define the main executable
ENTRYPOINT ["echo"]

# Define the default argument
CMD ["Hello Bhau!"]
```

Build and run:

```bash
docker build -t my-entrypoint .
docker run my-entrypoint
```

Output:

```text
Hello Bhau!
```

Now pass a different argument:

```bash
docker run my-entrypoint "Hello Docker!"
```

Output:

```text
Hello Docker!
```

The `ENTRYPOINT` remains `echo`, while the supplied argument replaces the default `CMD` argument.

**Real-world example:** Chai banavane is the main task. A customer asking for cutting chai or full chai changes the input, not the main task.

### D. Using CMD and ENTRYPOINT Together

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

# Define the main executable
ENTRYPOINT ["python", "app.py"]

# Set the default argument
CMD ["production"]
```

Example `app.py`:

```python
import sys

environment = sys.argv[1] if len(sys.argv) > 1 else "unknown"
print(f"Running in {environment} mode")
```

Build:

```bash
docker build -t my-python-app .
```

Run with the default argument:

```bash
docker run --rm my-python-app
```

Output:

```text
Running in production mode
```

Override the argument:

```bash
docker run --rm my-python-app testing
```

Output:

```text
Running in testing mode
```

**Remember:**

- `ENTRYPOINT` defines the main executable.
- `CMD` provides default arguments when used with an exec-form `ENTRYPOINT`.
- Runtime arguments replace the default `CMD` arguments in this example.

Prefer JSON/exec form, such as `CMD ["python", "app.py"]`, for predictable argument handling and signal delivery.

---

## 5. Practical Dockerfile 1 — Python Application

**Use case:** Run a simple Python application inside a container.

### Project Structure

```text
python-app/
├── Dockerfile
└── app.py
```

### Dockerfile

```dockerfile
# Use a lightweight Python base image
FROM python:3.12-slim

# Set the working directory
WORKDIR /app

# Copy the Python application into the image
COPY app.py .

# Run the application when the container starts
CMD ["python", "app.py"]
```

### app.py

```python
# Print a welcome message
print("Hello Bhau! My first Docker application.")
```

### Build and Run

```bash
docker build -t my-python-app .
docker run --name python-container my-python-app
```

Expected output:

```text
Hello Bhau! My first Docker application.
```

The container exits after the Python program finishes.

---

## 6. Practical Dockerfile 2 — Nginx Web Server

**Use case:** Serve a simple website through Nginx.

### Project Structure

```text
nginx-app/
├── Dockerfile
└── index.html
```

### Dockerfile

```dockerfile
# Use the official Nginx Alpine image
FROM nginx:alpine

# Copy our website into Nginx's default web directory
COPY index.html /usr/share/nginx/html/index.html

# Document the port used by the web server
EXPOSE 80
```

The official Nginx image already defines its startup command, so we do not need to add another `CMD`.

### index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Website</title>
</head>
<body>
    <h1>Hello Bhau!</h1>
    <p>My first website running inside Docker.</p>
</body>
</html>
```

### Build and Run

```bash
docker build -t my-nginx-app .
docker run -d --name web-container -p 8080:80 my-nginx-app
```

Open this URL in your browser:

`http://localhost:8080`

To stop and remove the container:

```bash
docker rm -f web-container
```

---

## 7. Practical Dockerfile 3 — Node.js Application

**Use case:** Run a simple Node.js application inside a container.

### Project Structure

```text
node-app/
├── Dockerfile
└── app.js
```

### Dockerfile

```dockerfile
# Use the Node.js Alpine base image
FROM node:22-alpine

# Set the working directory
WORKDIR /app

# Copy the application source code
COPY app.js .

# Run the Node.js application when the container starts
CMD ["node", "app.js"]
```

### app.js

```javascript
// Print a message to the terminal
console.log("Hello Bhau! Node.js is running in Docker.");
```

### Build and Run

```bash
docker build -t my-node-app .
docker run --name node-container my-node-app
```

Expected output:

```text
Hello Bhau! Node.js is running in Docker.
```

The container exits after the script finishes.

---

## 8. Best Practices for Writing Dockerfiles

1. Use a suitable, trusted base image.
2. Prefer small base images when appropriate.
3. Use `COPY` for ordinary file copying.
4. Use `.dockerignore` to exclude unnecessary files from the build context.
5. Combine related package installation commands where appropriate to reduce image layers.
6. Avoid storing passwords, tokens, or other secrets in the Dockerfile.
7. Prefer running applications as a non-root user when practical.
8. Use exec-form `CMD` and `ENTRYPOINT` for predictable behavior.
9. Pin or otherwise manage image versions appropriately for reproducible builds.
10. Keep the image as small and focused as practical.

### Example .dockerignore

```text
.git
.env
__pycache__
node_modules
```

Create this file in the build-context directory when those paths are relevant. Never copy secrets into an image.

---

## 9. Important Dockerfile Interview Questions

### Q1. What is a Dockerfile?

A Dockerfile is a text file containing instructions used to build a Docker image.

### Q2. What is the difference between RUN and CMD?

RUN executes commands during image building, whereas CMD specifies the default command or arguments when a container starts.

### Q3. What is the difference between CMD and ENTRYPOINT?

CMD provides the default command or arguments. ENTRYPOINT defines the main executable of the container. When used together in exec form, CMD commonly provides the default arguments to ENTRYPOINT.

### Q4. What is the purpose of FROM?

FROM specifies the base image used to build the Docker image.

### Q5. What is the purpose of WORKDIR?

WORKDIR sets the working directory for subsequent instructions.

### Q6. What is the difference between COPY and ADD?

COPY copies files and directories from the build context. ADD supports additional features, including extracting local tar archives.

### Q7. Does EXPOSE publish a port?

No. EXPOSE documents the intended listening port. Port publishing can be configured with `docker run -p`.

### Q8. How do you build a Docker image?

Use `docker build -t image-name .` from the directory containing the Dockerfile and required build-context files.

### Q9. How do you run a container from an image?

Use `docker run image-name`.

### Q10. What is .dockerignore?

It specifies files and directories to exclude from the Docker build context.

---

## 10. Quick Revision

| Instruction | Remember |
|---|---|
| FROM | Select the base image |
| WORKDIR | Set the working directory |
| COPY | Copy files into the image |
| RUN | Execute build-time commands |
| CMD | Set the default runtime command or arguments |
| ENTRYPOINT | Define the main executable |
| EXPOSE | Document the intended listening port |
| ENV | Set environment variables |
| ARG | Define build-time variables |
| USER | Set the user |
| VOLUME | Declare a mount point |

**Final interview answer:**

"A Dockerfile defines the instructions required to build a Docker image. It specifies the base image, application files, dependencies, working directory, and default startup behavior. Docker uses these instructions to create a reusable image from which containers can be started."
