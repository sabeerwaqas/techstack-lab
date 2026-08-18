# Docker Learning Guide

> My practical Docker learning notes — focused on the essential concepts required to start working with Kubernetes.

---

# 1. Why Do We Need Docker?

As software developers, we build applications that need to run in different environments:

- Development machine
- Another developer's machine
- Testing/QA environment
- Staging environment
- Production server
- Cloud infrastructure

An application doesn't depend only on its source code.

For example, a Java application may require:

```text
Application
    +
Java 17
    +
Specific Libraries
    +
Database
    +
Environment Variables
    +
Configuration
    +
Operating System Dependencies
```

If another developer has Java 21 instead of Java 17, a different database configuration, or a different operating system, the application may behave differently.

This is where we get the famous:

> **"It works on my machine."**

Docker helps solve this problem by packaging an application together with the environment and dependencies it needs to run.

---

# 2. Virtualization

One traditional solution to environment differences is **virtualization**.

A normal computer can be represented as:

```text
Physical Hardware
       ↓
Operating System
       ↓
Applications
```

With virtualization:

```text
Physical Hardware
       ↓
Host Operating System
       ↓
Hypervisor
       ↓
Virtual Machine
       ↓
Guest Operating System
       ↓
Application
```

A **Virtual Machine (VM)** behaves like a separate computer.

For example:

```text
Physical Machine
│
├── Windows (Host OS)
│
└── Hypervisor
      │
      ├── VM 1
      │     └── Ubuntu
      │          └── Application
      │
      └── VM 2
            └── Ubuntu
                 └── Application
```

## Problem With VMs

Every VM normally contains a complete guest operating system.

Therefore, running many VMs can consume significant:

- RAM
- CPU
- Storage
- Startup time

This can be unnecessarily heavy when all we really want is to run applications in isolated environments.

This leads us to **containerization**.

---

# 3. Containerization

Containerization provides a lighter approach.

Instead of creating a complete virtual computer for every application, we package the application and its dependencies into a **container**.

Conceptually:

```text
Host Operating System
       ↓
Container Runtime
       ↓
+-------------------+
| Container         |
|                   |
| Application       |
| Dependencies      |
| Configuration     |
+-------------------+
```

Multiple containers can run on the same operating system:

```text
Host OS
   │
   ├── Container A → Backend
   ├── Container B → PostgreSQL
   ├── Container C → Redis
   └── Container D → Frontend
```

Containers share the host operating system's kernel, which makes them generally lighter than traditional VMs.

---

# 4. Container vs Virtual Machine

### Virtual Machine

```text
Hardware
   ↓
Host OS
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

### Container

```text
Hardware
   ↓
Host OS
   ↓
Container Runtime
   ↓
Container
   ↓
Application
```

A useful mental model:

> **VM = Virtual Computer**

> **Container = Isolated Application Environment**

---

# 5. What Is Docker?

**Docker is a platform and collection of tools for building, running, sharing, and managing containerized applications.**

Docker makes containerization practical for developers.

Conceptually:

```text
Docker
│
├── Build Images
├── Run Containers
├── Manage Containers
├── Networking
├── Storage
└── Share Images
```

Docker is not the only container technology, but it is one of the most widely used tools in the container ecosystem.

---

# 6. Docker's Core Mental Model

The most important relationship to understand is:

```text
Dockerfile
     ↓
   Build
     ↓
 Docker Image
     ↓
    Run
     ↓
 Docker Container
```

Remember:

> **Dockerfile → Image → Container**

---

# 7. Docker Image

A **Docker image** is an immutable template/package used to create containers.

Think of an image as a blueprint.

```text
Docker Image
     │
     ├── Application
     ├── Runtime
     ├── Libraries
     ├── Dependencies
     └── Configuration
```

An image itself is not the running application. It is used to create containers.

One image can create multiple containers:

```text
             Docker Image
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
   Container   Container   Container
       1           2           3
```

This becomes important when scaling applications.

---

# 8. Docker Container

A **container** is a running or stopped instance created from an image.

```text
Image
  ↓
Container
```

For example:

```bash
docker run nginx
```

Docker uses the `nginx` image to create and run a container.

A container provides isolation for the application and its processes.

---

# 9. Image vs Container

| Image | Container |
|---|---|
| Blueprint/template | Instance created from the image |
| Immutable | Has a runtime lifecycle |
| Used to create containers | Runs the application |
| Can create many containers | Can be started/stopped/removed |

A useful analogy:

```text
Image
  ↓
Container
```

is similar to:

```text
Class
  ↓
Object
```

This is only an analogy, but it is useful for remembering the relationship.

---

# 10. Dockerfile

A **Dockerfile** is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM eclipse-temurin:17

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The basic workflow is:

```text
Dockerfile
     ↓
docker build
     ↓
Docker Image
     ↓
docker run
     ↓
Container
```

---

# 11. Essential Dockerfile Instructions

You do not need to memorize every Dockerfile instruction before learning Kubernetes.

Focus on these:

## FROM

Defines the base image.

```dockerfile
FROM eclipse-temurin:17
```

---

## WORKDIR

Sets the working directory inside the image/container.

```dockerfile
WORKDIR /app
```

---

## COPY

Copies files from the build context into the image.

```dockerfile
COPY target/app.jar app.jar
```

---

## RUN

Executes a command while building the image.

```dockerfile
RUN apt-get update
```

---

## EXPOSE

Documents the port the application is expected to use.

```dockerfile
EXPOSE 8080
```

> **Important:** `EXPOSE` does **not** publish the port to the host by itself.

---

## CMD

Provides a default command for the container.

```dockerfile
CMD ["java", "-jar", "app.jar"]
```

---

## ENTRYPOINT

Defines the main executable for the container.

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

For now, understand the basic difference between `CMD` and `ENTRYPOINT`. We can go deeper later if needed.

---

# 12. Building an Image

Once we have a Dockerfile, we can build an image:

```bash
docker build -t my-app:1.0 .
```

Meaning:

```text
docker build
     ↓
Read Dockerfile
     ↓
Build Image
     ↓
my-app:1.0
```

The `-t` option gives the image a name and tag.

---

# 13. Running a Container

To create and start a container:

```bash
docker run my-app:1.0
```

Docker will:

```text
Image
  ↓
Create Container
  ↓
Start Container
```

We can run multiple containers from the same image:

```text
my-app:1.0
    │
    ├── Container 1
    ├── Container 2
    └── Container 3
```

---

# 14. Essential Docker Commands

You do not need to memorize hundreds of Docker commands.

These are enough to get started.

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker version |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker run <image>` | Create and start a container |
| `docker stop <container>` | Stop a container |
| `docker start <container>` | Start a stopped container |
| `docker rm <container>` | Remove a container |
| `docker images` | List images |
| `docker rmi <image>` | Remove an image |
| `docker logs <container>` | View container logs |
| `docker exec -it <container> bash` | Execute a command inside a container |

---

# 15. Container Lifecycle

A basic container lifecycle looks like:

```text
             docker run
                 ↓
             Created
                 ↓
              Running
              ↙     ↘
        docker stop  docker restart
              ↓
             Stopped
              ↓
          docker rm
              ↓
            Removed
```

Understanding the lifecycle is more important than memorizing commands.

---

# 16. Docker Port Mapping

Containers have their own network environment.

Suppose an application inside the container listens on:

```text
8080
```

We may want to access it from our host machine.

Docker allows port mapping:

```text
Host Machine
localhost:8080
      │
      ↓
Docker Port Mapping
      │
      ↓
Container:8080
      │
      ↓
Application
```

Example:

```bash
docker run -p 8080:8080 my-app
```

The format is:

```text
-p HOST_PORT:CONTAINER_PORT
```

Example:

```bash
-p 9090:8080
```

means:

```text
Host Port 9090
       ↓
Container Port 8080
```

---

# 17. Environment Variables

Applications commonly need configuration such as:

```text
DATABASE_URL
DATABASE_USERNAME
DATABASE_PASSWORD
SPRING_PROFILES_ACTIVE
```

Docker allows us to pass environment variables to containers.

Example:

```bash
docker run \
  -e SPRING_PROFILES_ACTIVE=prod \
  my-app
```

We should avoid hardcoding environment-specific configuration into the application image.

This concept becomes very important in Kubernetes because Kubernetes provides mechanisms such as **ConfigMaps and Secrets**.

---

# 18. Docker Volumes

Containers are designed to be replaceable.

This creates a problem for applications that need persistent data.

For example:

```text
PostgreSQL Container
       ↓
     Data
```

If the container is removed, we don't want important database data to disappear.

Docker provides **volumes** for persistent storage.

Conceptually:

```text
Container
    │
    ↓
Docker Volume
    │
    ↓
Persistent Data
```

Example:

```bash
docker volume create postgres-data
```

Then:

```bash
docker run \
  -v postgres-data:/var/lib/postgresql/data \
  postgres
```

The exact PostgreSQL path depends on the image being used.

The important concept is:

> **Container lifecycle and persistent data lifecycle should be separated.**

This concept will become extremely important in Kubernetes.

---

# 19. Docker Networking

Containers often need to communicate with each other.

For example:

```text
Backend
   ↓
PostgreSQL
```

or:

```text
Backend
   ↓
Redis
```

Docker provides networking capabilities for this.

A simple example:

```text
Docker Network
│
├── backend
├── postgres
└── redis
```

Containers connected to the same Docker network can communicate with each other.

---

# 20. Docker Compose

Real applications often consist of multiple services.

For example:

```text
Frontend
    ↓
Backend
    ↓
PostgreSQL

Backend
    ↓
Redis
```

Managing each container manually can become inconvenient.

Docker Compose allows us to define multiple services together.

Conceptually:

```text
compose.yaml
     │
     ├── frontend
     ├── backend
     ├── postgres
     └── redis
```

A simplified Compose file:

```yaml
services:

  backend:
    image: my-backend:1.0
    ports:
      - "8080:8080"

  postgres:
    image: postgres
```

Common commands:

```bash
docker compose up
```

and:

```bash
docker compose down
```

Docker Compose is worth learning because it gives us practical experience running **multi-container applications**.

---

# 21. Docker Hub and Container Registries

We need a way to share Docker images.

A **container registry** stores and distributes container images.

Docker Hub is a popular public container registry.

The workflow looks like:

```text
Developer
    ↓
Build Image
    ↓
Docker Image
    ↓
Push
    ↓
Container Registry
    ↓
Pull
    ↓
Another Machine
```

For example:

```bash
docker pull nginx
```

downloads an image from a registry.

We can also push our own image:

```bash
docker push username/my-app:1.0
```

A registry becomes particularly important when we start working with Kubernetes.

---

# 22. Image Tags

Images commonly use tags to identify versions.

For example:

```text
my-app:1.0
my-app:1.1
my-app:2.0
```

The general format is:

```text
IMAGE_NAME:TAG
```

For example:

```text
nginx:1.27
```

Avoid relying blindly on:

```text
latest
```

for production deployments.

Explicit version tags make deployments more predictable.

---

# 23. The Complete Docker Workflow

At this point, we can understand the basic Docker workflow:

```text
                  Dockerfile
                      │
                      ↓
                docker build
                      │
                      ↓
                 Docker Image
                      │
              ┌───────┴───────┐
              ↓               ↓
         docker run       docker push
              ↓               ↓
          Container       Registry
              │               │
              ↓               ↓
         Application      docker pull
```

---

# 24. Example: Dockerizing a Spring Boot Application

Suppose we have:

```text
Spring Boot Application
       ↓
     Java 17
       ↓
     Port 8080
```

We create a Dockerfile:

```dockerfile
FROM eclipse-temurin:17

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build the image:

```bash
docker build -t my-spring-app:1.0 .
```

Run it:

```bash
docker run -p 8080:8080 my-spring-app:1.0
```

Now:

```text
Host
localhost:8080
      ↓
Docker
      ↓
Container
      ↓
Spring Boot
```

This is the level of Docker knowledge we need before moving toward Kubernetes.

---

# 25. What Docker Solves

Docker helps us package applications in a consistent way.

Instead of:

```text
"Install these 15 things before running my application."
```

we can work toward:

```text
Docker Image
     ↓
Container
     ↓
Application
```

The same image can be used across:

```text
Development
     ↓
Testing
     ↓
Staging
     ↓
Production
```

This improves:

- Portability
- Consistency
- Deployment reliability
- Environment isolation
- Developer onboarding

---

# 26. Docker vs Kubernetes

Docker and Kubernetes solve related but different problems.

### Docker

Docker helps us:

```text
Build
  ↓
Package
  ↓
Run
  ↓
Share
```

containerized applications.

### Kubernetes

Kubernetes helps us manage containerized workloads at scale.

For example:

```text
                    Kubernetes
                        │
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
     Deploy          Scale            Network
        ↓               ↓                ↓
    Containers      Replicas          Services
```

A simplified progression is:

```text
Docker
  ↓
Learn Containers
  ↓
Build Images
  ↓
Run Containers
  ↓
Push Images
  ↓
Kubernetes
  ↓
Manage Containers at Scale
```

---

# 27. Docker Knowledge Required Before Kubernetes

We do **NOT** need to master every Docker feature before learning Kubernetes.

## Must Know

- [x] Why containers exist
- [x] Containers vs Virtual Machines
- [x] What Docker is
- [x] Image vs Container
- [x] Dockerfile
- [x] `FROM`
- [x] `WORKDIR`
- [x] `COPY`
- [x] `RUN`
- [x] `EXPOSE`
- [x] `CMD`
- [x] `ENTRYPOINT`
- [x] `docker build`
- [x] `docker run`
- [x] Container lifecycle
- [x] `docker ps`
- [x] `docker logs`
- [x] `docker exec`
- [x] Port mapping
- [x] Environment variables
- [x] Basic networking
- [x] Basic volumes
- [x] Docker Compose
- [x] Docker Hub / container registries
- [x] Image tags
- [x] Push/Pull images

---

# 28. Docker Topics We Can Skip For Now

We should **not** spend excessive time learning Docker internals before Kubernetes.

These can be learned later when necessary:

- Docker Engine internals
- Linux namespaces in depth
- cgroups in depth
- Overlay filesystem internals
- Advanced Docker networking
- Advanced Docker security
- Rootless Docker internals
- Docker Swarm
- Advanced storage drivers
- Docker daemon internals
- Low-level container runtime implementation

These are useful topics, but they are **not prerequisites for starting Kubernetes**.

---

# 29. The 80/20 Docker Goal

Our goal is not:

> "Become a Docker expert."

Our goal is:

> **"Become comfortable enough with containers that Kubernetes concepts make sense."**

We should be able to:

```text
1. Take a Spring Boot application
            ↓
2. Create a Dockerfile
            ↓
3. Build an image
            ↓
4. Run a container
            ↓
5. Expose the application port
            ↓
6. Pass environment variables
            ↓
7. View logs
            ↓
8. Connect it to another container
            ↓
9. Persist data using a volume
            ↓
10. Push the image to a registry
```

Once we can do this comfortably:

> **Move to Kubernetes.**

Do not keep studying Docker indefinitely.

---

# 30. Docker → Kubernetes Roadmap

Our learning path:

```text
                    DOCKER
                       │
                       ↓
              Container Concepts
                       │
                       ↓
                Images & Dockerfile
                       │
                       ↓
                 Build & Run
                       │
                       ↓
           Ports & Environment Variables
                       │
                       ↓
             Networking & Volumes
                       │
                       ↓
                 Docker Compose
                       │
                       ↓
              Docker Registry
                       │
                       ↓
              ┌────────────────┐
              │  KUBERNETES 🚀 │
              └────────────────┘
                       │
                       ↓
                      Pod
                       │
                       ↓
                  Deployment
                       │
                       ↓
                    Service
                       │
                       ↓
                  ConfigMap
                       │
                       ↓
                    Secret
                       │
                       ↓
                   Volumes
                       │
                       ↓
                    Ingress
                       │
                       ↓
              Health Checks
                       │
                       ↓
             Resource Requests
             & Limits
                       │
                       ↓
                Rolling Updates
                       │
                       ↓
                 Autoscaling
```

---

# 31. Final Mental Model

Remember these relationships:

```text
Dockerfile
    │
    │ docker build
    ↓
Docker Image
    │
    │ docker run
    ↓
Docker Container
    │
    ├── Network
    ├── Environment Variables
    └── Volume
```

Images can be shared:

```text
Docker Image
     │
     │ docker push
     ↓
Container Registry
     │
     │ docker pull
     ↓
Another Machine
```

And Kubernetes eventually manages these containerized workloads:

```text
Docker Image
     ↓
Container
     ↓
Pod
     ↓
Deployment
     ↓
Kubernetes Cluster
```

---

# 32. Key Takeaways

### Docker

> A platform and set of tools for building, running, sharing, and managing containerized applications.

### Image

> An immutable template/package used to create containers.

### Container

> An isolated runtime environment created from an image.

### Dockerfile

> Instructions used to build a Docker image.

### Volume

> Persistent storage that exists independently from the container lifecycle.

### Network

> Allows containers and external systems to communicate.

### Registry

> A place where container images are stored and distributed.

### Docker Compose

> A tool for defining and managing multi-container applications.

### Kubernetes

> A platform for orchestrating and managing containerized workloads at scale.

---

# 33. The Most Important Sentence

If I remember only one thing from this Docker introduction:

> **Docker packages applications and their dependencies into portable container images, and containers provide isolated environments in which those applications can run consistently across different environments.**

The progression we are learning is:

```text
Problem
   ↓
"It works on my machine."
   ↓
Containerization
   ↓
Docker
   ↓
Image
   ↓
Container
   ↓
Registry
   ↓
Kubernetes
   ↓
Container Orchestration
```

---

# Next Step

Once I can comfortably use:

```bash
docker build
docker run
docker ps
docker stop
docker rm
docker logs
docker exec
docker images
docker pull
docker push
docker compose up
docker compose down
```

and understand:

```text
Image
Container
Dockerfile
Port
Environment Variable
Volume
Network
Registry
```

I am ready to move from:

**Docker Fundamentals → Kubernetes**
