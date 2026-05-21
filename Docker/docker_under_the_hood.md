Docker is basically a fancy orchestration layer around Linux kernel features. The containers themselves are *not* lightweight VMs — they’re isolated processes running on the same kernel as the host OS.

The magic comes from a handful of Linux kernel features working together.

---

# The Core Linux Kernel Features Docker Uses

## 1. Namespaces → Isolation

Linux **namespaces** make a process think it has its own private system.

Docker uses several namespaces:

| Namespace | What it isolates                   |
| --------- | ---------------------------------- |
| PID       | Process IDs                        |
| NET       | Network interfaces, ports, routing |
| MNT       | Filesystem mount points            |
| IPC       | Shared memory/message queues       |
| UTS       | Hostname/domain name               |
| USER      | User/group IDs                     |

Example:

Inside a container:

```bash
ps aux
```

might show:

```bash
PID 1 nginx
PID 7 worker
```

But on the host:

```bash
ps aux
```

those same processes may actually be:

```bash
PID 23841 nginx
PID 23852 worker
```

The container sees its own PID namespace.

Think of namespaces as giving each container its own illusion of a separate machine.

---

## 2. cgroups → Resource Limits

Namespaces isolate visibility.

But isolation alone doesn’t stop a container from eating all CPU or RAM.

That’s where **control groups (cgroups)** come in.

Docker uses cgroups to limit:

* CPU usage
* Memory
* Disk I/O
* Network bandwidth
* Number of processes

Example:

```bash
docker run --memory=512m --cpus=1 nginx
```

This tells the kernel:

* max RAM = 512 MB
* max CPU = 1 core

Without cgroups, one bad container could freeze the whole machine.

---

## 3. Union Filesystems → Layered Images

Docker images are made from stacked filesystem layers.

Linux supports this using filesystems like:

* OverlayFS
* AUFS (older)
* Btrfs
* ZFS

Docker commonly uses **OverlayFS** today.

Example:

You build an image:

```Dockerfile
FROM ubuntu
RUN apt install python3
COPY app.py .
```

Each step becomes a layer.

The kernel merges them into one virtual filesystem.

Benefits:

* efficient storage
* shared layers
* fast image downloads
* copy-on-write behavior

If 100 containers use Ubuntu, the base layer exists only once on disk.

---

## 4. chroot / pivot_root → Filesystem Isolation

Containers need their own root filesystem.

Linux provides mechanisms like:

* `chroot`
* `pivot_root`

Docker uses these to make a process believe:

```bash
/
```

starts from the container image filesystem.

So the container cannot normally see the host filesystem.

---

## 5. Capabilities → Fine-Grained Privileges

Linux traditionally had:

* root
* non-root

Too coarse.

Linux capabilities split root powers into smaller permissions.

Examples:

* `CAP_NET_ADMIN`
* `CAP_SYS_ADMIN`
* `CAP_SYS_TIME`

Docker removes many dangerous capabilities by default.

So even if a process runs as root *inside* the container, it’s heavily restricted.

This is a huge security feature.

---

## 6. Seccomp → Syscall Filtering

Processes talk to the kernel using **system calls**.

Docker uses Linux **seccomp** filters to block dangerous syscalls.

Example blocked actions:

* kernel module loading
* low-level debugging calls
* certain mount operations

This reduces attack surface.

---

## 7. AppArmor / SELinux → Mandatory Access Control

Docker can integrate with Linux security frameworks:

* AppArmor
* SELinux

These enforce additional policies like:

* “this container cannot access these files”
* “this process cannot open raw sockets”

Even if the process is compromised.

---

# What Happens When You Run a Container?

When you do:

```bash
docker run nginx
```

roughly this happens:

1. Docker daemon talks to container runtime (`containerd`, `runc`)
2. Kernel creates namespaces
3. Kernel creates cgroups
4. OverlayFS mounts image layers
5. Network namespace + virtual ethernet setup
6. Process starts inside isolated environment
7. Seccomp/AppArmor rules applied

At the end:

The container is just a Linux process with special kernel settings.

Not a VM.

---

# Containers vs Virtual Machines

| Feature         | Containers         | VMs             |
| --------------- | ------------------ | --------------- |
| Kernel          | Shared host kernel | Separate kernel |
| Boot speed      | Seconds/ms         | Minutes         |
| Isolation       | Process-level      | Hardware-level  |
| Memory overhead | Low                | High            |
| OS flexibility  | Same kernel family | Any OS          |

A Linux container cannot run a Windows kernel directly.

Because containers share the host kernel.

---

# Important Insight

Docker itself is not the core innovation.

Most underlying technologies existed in Linux for years:

* namespaces (2002+)
* cgroups (2007)
* chroot (1979)

Docker’s innovation was:

* packaging
* developer UX
* image distribution
* standardization
* easy orchestration

It made kernel features practical and easy to use.

---

# Architecture Stack

A simplified view:

```text
+-------------------+
| Docker CLI        |
+-------------------+
| Docker Daemon     |
+-------------------+
| containerd        |
+-------------------+
| runc              |
+-------------------+
| Linux Kernel      |
| namespaces        |
| cgroups           |
| seccomp           |
| OverlayFS         |
+-------------------+
| Hardware          |
+-------------------+
```

---

# Tiny Mental Model

You can think of a Docker container as:

> “A regular Linux process wrapped in kernel-enforced isolation and resource controls.”

That’s the whole idea.
