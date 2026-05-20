# Docker Concepts

## Images Build Upon Other Images

Think of images like inheritance — very similar to class hierarchies:

```
Base image → Derived image → More specialized image
```

For example:

```
ubuntu
  └── python image
        └── your flask app image
```

In reality, many official images are already built this way. For example, Python images often internally start from Debian or Alpine Linux images.

In Docker, **images are layered on top of other images**.

**Example:**

```dockerfile
FROM ubuntu
RUN apt update
RUN apt install -y nginx
CMD ["nginx", "-g", "daemon off;"]
```

Here's what's happening:

- `ubuntu` is the base image
- Your new image adds extra layers:
  - package updates
  - nginx installation
  - startup command

So your custom image becomes:

```
ubuntu image
   +
nginx installation
   +
your config
   =
new image
```

That layering system is one of Docker's biggest ideas.

---

## Images Are Immutable Layers

Every Docker instruction creates a new layer:

```dockerfile
FROM ubuntu        # layer 1
RUN apt update     # layer 2
RUN apt install    # layer 3
COPY . /app        # layer 4
```

Docker caches these layers to make rebuilds fast.

### Common Base Images

People often start from:

- Ubuntu
- Alpine Linux
- Debian
- Node.js official image
- Python official image
- NGINX image

**Example:**

```dockerfile
FROM python:3.12
```

That already contains Python installed.

---

## Docker Image Layers

A Docker image layer is essentially:

> "a filesystem snapshot / filesystem diff over the previous layer"

So this:

```dockerfile
FROM python:3.12
```

already gives you a complete filesystem containing:

- Linux base OS files
- libraries
- Python binaries
- package managers
- configs
- etc.

And then:

```dockerfile
RUN apt update
```

creates another layer containing filesystem changes relative to the previous one.

### Important Nuance

`apt update` mainly updates package index files, like:

```
/var/lib/apt/lists/*
```

So that layer mostly contains updated metadata files.

But:

```dockerfile
RUN apt install -y curl
```

would add:

- `/usr/bin/curl`
- shared libraries
- dependency files
- package metadata

So that new layer becomes a filesystem diff containing those additions.

### Visualizing Layers

```
Layer 3  ->  install curl
Layer 2  ->  apt update
Layer 1  ->  python 3.12 environment
Layer 0  ->  ubuntu/debian base filesystem
```

Docker combines them into one unified filesystem using a **union filesystem** mechanism (like OverlayFS on Linux). To the container, it looks like one normal filesystem.

### Another Important Thing

When you do:

```dockerfile
RUN rm bigfile.txt
```

the file is hidden in the new layer, but it may **still physically exist in older layers**.

That's why image size can stay large even after deleting files later.

**Example:**

```dockerfile
RUN apt install huge-package
RUN apt remove huge-package
```

This can still produce a large image because the installation layer still exists underneath.

That's why people combine commands:

```dockerfile
RUN apt update && \
    apt install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

Everything happens in one layer, so unnecessary files never get preserved in intermediate layers.

### Fun Fact: Containers Add One More Writable Layer

Images are **read-only**.

When you run a container:

```bash
docker run ubuntu
```

Docker adds:

```
read-only image layers
+
thin writable container layer
```

So changes made while the container runs go into that temporary writable layer.

---

## Examples of Commands

### `docker run hello-world`

```
docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

---

### `docker run -it ubuntu bash`

One of the most common ways to start an interactive container in Docker.

**Breakdown:**

| Part     | Meaning                                                                                      |
|----------|----------------------------------------------------------------------------------------------|
| `docker` | The Docker CLI tool. You're telling Docker to do something.                                  |
| `run`    | Create and start a new container from an image.                                              |
| `-it`    | Combination of two flags: `-i` and `-t` for interactive terminal use.                        |
| `ubuntu` | The image name. Docker pulls the Ubuntu image if it isn't already local.                     |
| `bash`   | The command to run inside the container. Here, it starts the Bash shell.                     |

**The `-it` flags in detail:**

**`-i` (interactive)**
- Keeps STDIN open so you can type commands into the container.
- Without it, the shell may immediately exit because no input is attached.

**`-t` (TTY terminal)**
- Allocates a terminal interface so the shell behaves like a normal Linux terminal.
- Without it, output looks less friendly and interactive shell features may not work properly.

**Together, `-it` gives you a proper interactive shell session.**