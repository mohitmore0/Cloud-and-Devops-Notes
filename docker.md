## 1. Core Concepts (Rapid Recall)

- **Docker**: open-source platform to build, ship, and run apps in containers using OS-level virtualization (shares host Linux kernel — no full guest OS).
- **Image vs Container**: Image = read-only template (blueprint). Container = a running, writable instance of that image.
- **Why Docker**: solves "works on my machine" — packages app + dependencies + config into one portable unit.
- **Written in**: Go. Engine runs natively on Linux; other OSes use a Linux VM under the hood.

### Docker vs Virtual Machine

| Docker (Container) | Virtual Machine |
|---|---|
| Shares host OS kernel | Full guest OS per VM |
| Starts in seconds | Starts in minutes |
| Lightweight (MBs) | Heavy (GBs) |
| Less isolation, more density | Strong isolation, less density |

### Pros / Cons (say both when asked "why Docker")

- **Pros**: no RAM pre-allocation, CI efficiency (same image every stage), cheap, lightweight, runs anywhere (bare metal/VM/cloud), reusable images, fast container creation.
- **Cons**: not ideal for rich-GUI apps, hard to manage at scale without orchestration (→ Kubernetes), no native cross-platform image portability (Windows image ≠ Linux image), no built-in backup/DR for data — use volumes + external backup.

### Architecture

- **Client** → sends commands (build/pull/run) via CLI or REST API.
- **Docker Host** → runs the **Docker Daemon (dockerd)**, which manages images, containers, networks, storage.
- **Registry** → stores images (Docker Hub = public, private registry = internal/enterprise).

---

## 2. Key Components

- **Docker Daemon**: background process on host; runs containers, manages services; daemons can talk to other daemons.
- **Docker Client**: CLI/API used to talk to the daemon; one client can talk to multiple daemons.
- **Docker Host**: environment providing daemon + images + containers + network + storage.
- **Registry (Hub)**: Public (Docker Hub) vs Private (internal image sharing).
- **Image**: read-only, layered, built from a Dockerfile or committed from a container.
- **Container**: a running (or stopped) instance created from an image; think of image = class, container = object.

---

## 3. Must-Know Commands

### Images

| Task | Command |
|---|---|
| List local images | `docker images` |
| Search Docker Hub | `docker search <image_name>` |
| Pull image | `docker pull <image_name>` |
| Build image from Dockerfile | `docker build -t <image_name>:<tag> .` |
| Delete image | `docker image rm <image_name>` |
| Commit container → image | `docker commit <container_name> <new_image_name>` |

### Containers

| Task | Command |
|---|---|
| Run + name + interactive | `docker run -it --name <container_name> <image_name> /bin/bash` |
| List running containers | `docker ps` |
| List all containers | `docker ps -a` |
| Start / Stop | `docker start <container_name>` / `docker stop <container_name>` |
| Attach vs Exec (new shell) | `docker attach <container_name>` / `docker exec -it <container_name> /bin/bash` |
| Remove container | `docker rm <container_name>` |
| Inspect container | `docker container inspect <container_name>` |
| Diff vs base image | `docker diff <container_name>` — A=Add, C=Change, D=Delete |

### Volumes

| Task | Command |
|---|---|
| List / create / remove | `docker volume ls` / `docker volume create <volume_name>` / `docker volume rm <volume_name>` |
| Prune unused volumes | `docker volume prune` |
| Inspect volume | `docker volume inspect <volume_name>` |
| Run with named volume | `docker run -it --name c1 -v /myvolume <image> /bin/bash` |
| Share volume container → container | `docker run -it --name c2 --privileged=true --volumes-from c1 <image> /bin/bash` |
| Bind mount host → container | `docker run -it --name c1 -v /host/path:/container/path <image> /bin/bash` |

### Service

| Task | Command |
|---|---|
| Check / start docker service | `service docker status` / `service docker start` |

---

## 4. Dockerfile Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Base image — must be the first instruction. |
| `RUN` | Executes a command; creates a new image layer. |
| `MAINTAINER` | Author / owner / description (legacy). |
| `COPY` | Copies files from local build context only (no URLs). |
| `ADD` | Like COPY, but can fetch from a URL and auto-extract archives. |
| `EXPOSE` | Documents a port the container listens on (e.g. 8080, 80). |
| `WORKDIR` | Sets the working directory inside the container. |
| `ENV` | Sets environment variables, persists at container runtime. |
| `ARG` | Build-time variable only; **NOT** available inside the running container. |
| `CMD` | Default command at container start; overridable at `docker run`. |
| `ENTRYPOINT` | Like CMD but higher priority — always runs first; harder to override. |

### Example minimal Dockerfile

```dockerfile
FROM ubuntu
WORKDIR /tmp
RUN echo "Hello World" > /tmp/testfile
ENV myname DevOps_Brother
COPY testfile /tmp
ADD test.tar.gz /tmp
```

Build → run:

```bash
docker build -t <image_name> .
docker run -it --name <container_name> <image_name> /bin/bash
```

---

## 5. Volumes — Key Interview Points

- A volume is a directory that persists data outside the container's writable layer.
- Declared only at container creation time — can't add a volume to an already-running container.
- Survives container stop/delete; NOT included when you rebuild/update an image.
- Two mapping types: **Container ↔ Container** (`--volumes-from`) and **Host ↔ Container** (`-v host_path:container_path`).
- Benefit: decouples storage from the container lifecycle — enables sharing data across containers.

---

## 6. EXPOSE vs PUBLISH (-p) — Classic Trick Question

| Option | Accessible from |
|---|---|
| Neither EXPOSE nor -p | Only inside the container itself |
| EXPOSE only | Other Docker containers (inter-container), not from outside Docker |
| EXPOSE + -p | Anywhere — even outside Docker (public access) |

> **Note:** if you use `-p` without EXPOSE, Docker implicitly exposes the port too (public ⇒ automatically inter-container reachable).

---

## 7. Rapid-Fire Interview Q&A

**Q: Difference between docker attach and docker exec?**
A: `attach` connects your terminal's stdio to the container's existing main process (no new process). `exec` starts a brand-new process (e.g. a new shell) inside a running container — the standard way to "get inside" a container for debugging.

**Q: What happens to a volume when you delete its container?**
A: The volume is NOT deleted — it's decoupled from the container lifecycle and can be reattached to a new container.

**Q: Can you create a volume on an already-running container?**
A: No — a directory can only be declared as a volume at container-creation time.

**Q: ENV vs ARG?**
A: ENV values persist and are accessible inside the running container. ARG is build-time only — once the image is built, the running container can't access ARG values.

**Q: CMD vs ENTRYPOINT?**
A: Both define the default startup command, but ENTRYPOINT has higher priority and always executes; CMD is easily overridden by arguments passed to `docker run`.

**Q: Why can't COPY fetch from a URL but ADD can?**
A: COPY is intentionally restricted to the local build context for predictability/security; ADD additionally supports remote URLs and auto-extracts local tar archives.

**Q: How do you make a container's port publicly reachable?**
A: Use `-p host_port:container_port` at `docker run` (implicitly also exposes it to other containers).

**Q: How to see what changed between a container and its base image?**
A: `docker diff <container_name>` — shows A (added), C (changed), D (deleted) paths.

**Q: Docker's main limitation at scale?**
A: Managing large numbers of containers manually is hard — this is exactly the problem orchestrators like Kubernetes/Docker Swarm solve.

---

## 8. Scenario-Based Questions

**Scenario:** You stop and remove a container. The developer says all their data is gone. What went wrong and how would you prevent it?
A: Data written to a container's writable layer disappears when the container is removed. Prevent it by mounting a volume or bind mount (`-v`) at container creation so data lives outside the container's lifecycle.

**Scenario:** Two containers on the same host need to share the same live data (e.g. logs) and see each other's updates instantly. How do you set this up?
A: Use container ↔ container volume sharing: create the volume on one container, then run the second with `--volumes-from <container_name> --privileged=true` so both read/write the same volume.

**Scenario:** A service inside your container works fine when tested with `docker exec`, but no one outside the host can reach it. What's missing?
A: The port hasn't been published. EXPOSE alone only makes it reachable from other containers, not the outside world — add `-p host_port:container_port` at `docker run` so it's publicly accessible.

**Scenario:** You update your Dockerfile and rebuild the image, but the new container is missing files a teammate added to a running container earlier. Why?
A: Manual changes inside a running container aren't part of the image unless committed (`docker commit`) or explicitly added via Dockerfile (COPY/ADD/RUN). Rebuilding from the Dockerfile only reflects what's written there — data on volumes is also excluded from image rebuilds.

**Scenario:** You need to pass a secret build-time API key used only during `docker build`, but you don't want it accessible once the container is running. Which instruction do you use?
A: `ARG` — it's available only at build time and is not retained in the running container, unlike `ENV` which persists at runtime.

**Scenario:** You SSH into a host, run `docker ps`, and see nothing — but `docker ps -a` shows several containers. What does this tell you, and what's your next step?
A: All containers exist but are stopped (not running). Next step: check logs (`docker logs <name>`) and inspect (`docker container inspect <name>`) to find why they exited, then `docker start <name>` if appropriate.

**Scenario:** Your teammate wants to debug a live production container without disturbing its main process. What command do you use and why?
A: `docker exec -it <container_name> /bin/bash` — it opens a new, independent process (a debug shell) inside the container instead of attaching to (and risking interference with) the main running process like `docker attach` would.

**Scenario:** You deleted an image with `docker image rm`, but disk space didn't free up much, and old dangling volumes are piling up. What commands help clean up?
A: Use `docker volume prune` for unused volumes, and more broadly `docker system prune` (images/containers/networks) — always confirm nothing important depends on them first, since volumes aren't auto-deleted with containers by design.

**Scenario:** An interviewer asks: "Would you use Docker for a desktop GUI application with heavy user interaction?" How do you answer?
A: Generally no — Docker is optimized for server-side, headless, portable workloads; rich-GUI apps are a known weak spot, and a VM or native install is usually a better fit.

---

> **Tip:** In interviews, always pair a definition with a *why* (e.g. "volumes exist because container writable layers are ephemeral") — it signals real understanding, not memorization.
