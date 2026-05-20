# Docker: Theoretical Concepts and Architecture

## 1. What is Docker?

Docker is a platform that packages applications and their dependencies into lightweight, portable containers. A container is an isolated native **process** that runs your application with everything it needs, sharing the host's kernel but maintaining its own filesystem, process tree, and network stack.

### Key Benefits
- **Lightweight**: Containers share the host kernel, making them much smaller than virtual machines
- **Portable**: Build once, run anywhere consistently across different environments
- **Isolated**: Each container runs independently without interfering with others
- **Efficient**: Fast startup times and minimal resource overhead
- **Consistency**: Ensures "build once, run anywhere" across development, testing, and production

## 2. Why Containers?

Containers are a lightweight form of virtualization that package an application and its dependencies together. They provide consistency across different environments, making it easier to develop, test, and deploy applications.

### Why Containers in the Cloud?

**Microservices Architecture**
- Enables breaking down applications into smaller, manageable services
- Each service runs in its own container

**Continuous Integration / Continuous Deployment (CI/CD)**
- Containers ensure "build once, run anywhere"
- Streamlines automated testing and deployment pipelines

**Application Modernization**
- Legacy applications are containerized to move them to the cloud
- Enables gradual migration to modern architectures

**Hybrid and Multi-Cloud Deployments**
- The same container image can run:
  - On-premises (physically hosts)
  - In private cloud
  - Across multiple public clouds (AWS, Azure, GCP)

## 3. Docker Architecture

Docker uses a **client-server architecture** with the following components:

### Docker Client
The CLI you interact with using the `docker` command. It sends commands to the Docker daemon.

### Docker Daemon (dockerd)
The background service that performs the actual work:
- Manages containers (create, start, stop, remove)
- Builds images from Dockerfiles
- Handles networks and network configuration
- Manages storage volumes
- Communicates with the Docker CLI
- Pulls/pushes images from/to registries

### Workflow Example: `docker run hello-world`
1. The Docker client contacts the Docker daemon
2. The Docker daemon pulls the "hello-world" image from Docker Hub (if not present locally)
3. The Docker daemon creates a new container from that image
4. The Docker daemon executes the container's command
5. The Docker daemon streams the output to the Docker client, which displays it in your terminal

## 4. How Docker Works Under the Hood

### Linux Kernel Features

Docker leverages several Linux kernel features for containerization:

**Namespaces** → Provide isolation for:
- **PID (Process ID)**: Process tree isolation - container sees only its own processes
- **NET (Network)**: Network stack isolation - own interfaces, routing tables, firewall rules
- **MNT (Mount)**: Filesystem isolation - own mount points
- **IPC (Inter-Process Communication)**: Isolation for shared memory, semaphores, message queues
- **UTS (Unix Timesharing System)**: Hostname and domain name isolation
- **USER**: User and group ID isolation

**cgroups (Control Groups)** → Limit and control resource allocation:
- CPU usage and scheduling
- Memory allocation and limits
- Disk I/O bandwidth
- Network bandwidth
- Device access

**Union Filesystems (OverlayFS)** → Enable layered container images:
- Multiple read-only layers stacked together
- Single writable layer on top
- Efficient storage through layer reuse

**Capabilities & seccomp** → Apply security constraints:
- Fine-grained privilege control
- System call filtering

### What's Shared vs Isolated?

**✓ Shared with Host:**
- Linux Kernel
- Kernel modules
- System call interface
- Device drivers

**✗ Isolated per Container:**
- Filesystem (each container has its own root filesystem)
- Process tree (PID namespace)
- Network stack (own interfaces, routing, firewall)
- IPC mechanisms
- Hostname
- User accounts

## 5. Core Concepts

### Docker Images

A **Docker image** is a read-only template containing an application and all its dependencies, including the code, runtime, libraries, environment variables, and configurations. It acts as a blueprint for creating Docker containers.

**Key Characteristics:**

**Layered Structure**
- Docker images are built in layers
- Each layer represents a change or instruction in the Dockerfile
- Each instruction (FROM, RUN, COPY, etc.) creates a new layer
- Layers are stacked on top of each other
- This layering enables efficient storage and reuse

**Immutable**
- Once an image is built, it cannot be changed
- This ensures consistency and reproducibility
- Any modification creates a new image

**Portable**
- Images are designed to be portable
- Build an image once and run it consistently across different environments
- Works on local machines, cloud platforms, or other servers

**Template for Containers**
- An image is the foundation from which Docker containers are launched
- When you run an image, it becomes a running instance called a container

**Stored in Registries**
- Docker images are typically stored in registries
- Registries serve as central repositories for sharing and distributing images
- Examples: Docker Hub, GitHub Container Registry, GitLab Container Registry

**Size Comparison:**
- Alpine container: ~5 MB (minimal BusyBox, musl libc, apk package manager)
- Ubuntu container: ~70-90 MB (bash, core utilities, standard libraries)
- Full Ubuntu OS: ~3-5 GB (includes kernel, drivers, systemd, system services)

### Docker Containers

A **Docker container** is a running instance of an image. It's the actual execution environment.

**What Containers Include:**
- Minimal userspace required for the application
- Application binaries and executable files
- Required libraries and dependencies
- Configuration files
- Runtime environment

**What Containers Do NOT Include:**
- Kernel (containers use the host kernel)
- Hardware drivers
- System services (cron, systemd, udev) unless explicitly added
- Init system (usually not required)

**Container Filesystem:**
- Uses image layers (shared across containers, read-only)
- Has its own writable layer for runtime changes
- Extremely lightweight due to layer reuse across multiple containers
- Changes in the writable layer are lost when container is removed (unless using volumes)

**Redundancy and Efficiency:**
- No need to worry about redundancy in the local registry
- Docker reuses image layers if they are already present
- Multiple containers from the same image share the base layers
- Only the writable layer is unique per container

### Dockerfile

A **Dockerfile** is the blueprint of an image. It's a text file containing instructions for building a Docker image. Each instruction creates a layer in the final image.

### Docker Registry

A **Docker registry** is a remote server that stores and distributes Docker images:

**Public Registries:**
- **Docker Hub**: Official public registry (hub.docker.com)
- **GitHub Container Registry**: GitHub's image hosting (ghcr.io)
- **GitLab Container Registry**: GitLab's image hosting

**Private Registries:**
- Self-hosted solutions
- Enterprise registries (AWS ECR, Azure ACR, Google GCR)
- On-premises registries for organizations

## 6. Container Orchestration

In real cloud environments, containers are rarely run individually. Instead, they are managed by container orchestration platforms.

### Managing Containers at Scale

**Container Management in the Cloud:**
1. Build the Container Image
2. Store the Container Image in a cloud Registry
3. Deploy Containers on Cloud Infrastructure

### Kubernetes

The most common orchestration platform for managing containers at scale.

**Cloud-Managed Kubernetes Services:**
- **Amazon EKS** (Elastic Kubernetes Service)
- **Azure AKS** (Azure Kubernetes Service)
- **Google GKE** (Google Kubernetes Engine)

**Kubernetes Capabilities:**
- Automatic container deployment and scaling
- Load balancing and service discovery
- Self-healing (automatic restart of failed containers)
- Rolling updates and rollbacks
- Resource management and scheduling
- Configuration and secret management

## 7. Networking Concepts

### Container Isolation
- Containers are isolated from the host and each other by default
- Each container has its own network namespace
- Containers cannot directly access each other without explicit configuration

### Host-Container Communication
- Containers can access the outer world (including localhost) via the host machine
- The outer world cannot access containers unless ports are explicitly mapped
- Port mapping uses the `-p` flag to expose container ports to the host

## 8. Best Practices and Principles

**Image Design:**
- Use specific base image tags (avoid `latest`) for reproducibility
- Minimize layers by combining RUN commands
- Keep images small by using Alpine-based images
- Use .dockerignore to exclude unnecessary files

**Security:**
- Run containers as non-root user for security
- Apply principle of least privilege
- Keep base images updated

**Architecture:**
- One process per container (follow microservices pattern)
- Separate concerns (database, application, cache in different containers)
- Use volumes for persistent data

**Operations:**
- Implement health checks for production containers
- Use environment variables for configuration
- Log to stdout/stderr for container logs

## 9. Key Terminology

- **Image**: Read-only template (blueprint)
- **Container**: Running instance of an image
- **Dockerfile**: Text file with instructions to build an image
- **Layer**: Each instruction in Dockerfile creates a layer
- **Registry**: Server that stores and distributes images
- **Tag**: Version label attached to an image name
- **Volume**: Persistent storage mechanism for containers
- **Port Mapping**: Exposing container ports to the host
- **Daemon**: Background service (dockerd) that manages Docker