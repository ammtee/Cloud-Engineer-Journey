# Docker Basics

## What Docker Is

Docker packages an application with everything it needs to run (code, runtime, libraries, system tools) into a single unit called a **container**. Containers run consistently across any environment — solving the classic "it works on my machine" problem.

## Containers vs. Virtual Machines

| | Container | Virtual Machine |
|---|---|---|
| What's virtualized | The OS user space (shares host kernel) | Full hardware + OS |
| Startup time | Seconds (often milliseconds) | Minutes |
| Size | MBs | GBs |
| Isolation | Process-level | Full OS-level |
| Density | Many per host | Few per host |

Containers are lighter and faster because they don't run a full guest OS — they share the host's kernel while isolating processes, filesystem, and network.

## Core Concepts

| Concept | Description |
|---|---|
| **Image** | A read-only template containing the application and its dependencies |
| **Container** | A running (or stopped) instance of an image |
| **Dockerfile** | Instructions for building an image |
| **Registry** | Where images are stored/shared (e.g., Docker Hub, Amazon ECR) |
| **Volume** | Persistent storage that survives beyond a container's lifecycle |
| **Network** | Virtual networking connecting containers to each other and the host |

## Essential Commands

```bash
docker --version                  # check Docker is installed
docker pull nginx                 # download an image from a registry
docker images                     # list local images
docker run -d -p 8080:80 nginx    # run a container in detached mode, mapping port 8080→80
docker ps                         # list running containers
docker ps -a                      # list all containers, including stopped
docker stop <container_id>        # stop a running container
docker start <container_id>       # start a stopped container
docker rm <container_id>          # remove a container
docker rmi <image_id>             # remove an image
docker logs <container_id>        # view container logs
docker exec -it <container_id> bash  # open an interactive shell inside a running container
```

## Building and Tagging Images

```bash
docker build -t my-app:1.0 .          # build an image from a Dockerfile in the current directory
docker tag my-app:1.0 my-app:latest   # add another tag to an existing image
docker push myregistry/my-app:1.0     # push to a registry (after docker login)
```

## Best Practices

- Use official base images from trusted sources (`node:20-alpine`, not random unofficial images)
- Prefer `-alpine` or `-slim` variants to keep image size small
- Never store secrets inside an image — use environment variables or a secrets manager at runtime
- Use `.dockerignore` to exclude unnecessary files (like `.git`, `node_modules`) from the build context
- Run containers as a non-root user where possible for better security
- Clean up unused images/containers regularly (`docker system prune`)

## Interview Prep

**Q: What's the difference between a Docker image and a container?**
An image is a read-only, immutable template — think of it like a class definition. A container is a running (or stopped) instance created from that image — think of it like an object instantiated from that class. You can run multiple containers from the same image.

**Q: Why are containers faster to start than virtual machines?**
Containers share the host machine's kernel and only virtualize the user space, so starting one just means starting a process with isolated resources — no OS boot required. VMs virtualize the entire hardware stack and have to boot a complete guest operating system, which takes significantly longer.

**Q: How would you persist data used by a container beyond its lifecycle?**
Use a Docker **volume** (or a bind mount) — data written to a volume lives outside the container's writable layer, so it survives even if the container is removed and a new one is started from the same image, mounting the same volume.

**Q: Why avoid running processes as root inside a container?**
If an attacker compromises the application inside the container, running as root gives them root-level access within that container's namespace, increasing the potential impact if there's ever a container escape vulnerability. Running as a non-root user limits the blast radius, following the same least-privilege principle used in IAM.
