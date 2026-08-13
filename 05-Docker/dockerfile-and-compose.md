# Dockerfile & Docker Compose

## Dockerfile Basics

A Dockerfile is a text file of instructions Docker follows to build an image, layer by layer.

```dockerfile
# Example: a simple Node.js app
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 3000

USER node

CMD ["node", "server.js"]
```

## Key Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | The base image to build on top of |
| `WORKDIR` | Sets the working directory inside the container |
| `COPY` | Copies files from the host into the image |
| `RUN` | Executes a command at build time (e.g., installing dependencies) |
| `EXPOSE` | Documents which port the container listens on (doesn't actually publish it) |
| `ENV` | Sets an environment variable inside the image |
| `USER` | Sets the user the container runs as (avoid running as root) |
| `CMD` | The default command run when the container starts |
| `ENTRYPOINT` | Similar to `CMD`, but harder to override — often combined with `CMD` for default args |

## Layer Caching

Each instruction creates a new image layer. Docker caches layers and reuses them if nothing changed — this is why `COPY package*.json ./` followed by `RUN npm install` happens *before* `COPY . .`: dependency installation only re-runs when `package.json` actually changes, not on every code edit.

## Multi-Stage Builds

Used to keep final images small by separating build tools from the runtime image:

```dockerfile
# Stage 1: build
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Stage 2: run (only the build output, no dev dependencies)
FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

## Docker Compose

Compose defines and runs **multi-container** applications using a single YAML file — instead of manually running multiple `docker run` commands.

```yaml
# docker-compose.yml
version: "3.9"

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=example
      - MYSQL_DATABASE=appdb
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

## Compose Commands

```bash
docker compose up              # start all services (foreground)
docker compose up -d            # start all services (detached)
docker compose down             # stop and remove containers/networks
docker compose down -v          # also remove named volumes
docker compose ps               # list running services
docker compose logs -f web      # follow logs for a specific service
docker compose build            # rebuild images defined in the compose file
```

## Networking Between Containers

Compose automatically creates a shared network for all services in the file — containers can reach each other **by service name** (e.g., the `web` service connects to the database using host `db`, not `localhost`), which is exactly how the example above works.

## Best Practices

- Order Dockerfile instructions from least-to-most frequently changing, to maximize cache reuse
- Use multi-stage builds to keep production images lean (no build tools/dev dependencies in the final image)
- Never hardcode secrets in a Dockerfile or `docker-compose.yml` — use environment variables injected at runtime, or Docker secrets
- Pin base image versions (`node:20-alpine`, not just `node:latest`) for reproducible builds
- Use named volumes for anything that needs to persist (databases), not container-local storage

## Interview Prep

**Q: Why does instruction order matter in a Dockerfile?**
Docker caches each layer and only rebuilds layers from the point where something actually changed. By copying dependency manifests (`package.json`) and installing dependencies *before* copying the rest of the application code, code changes don't force a full dependency reinstall on every build — only actual dependency changes do.

**Q: What problem does a multi-stage build solve?**
It lets you use a full-featured image with build tools (compilers, dev dependencies) to build the application, then copy only the final build artifacts into a much smaller runtime image — keeping the production image lean and reducing its attack surface, without needing a separate build pipeline.

**Q: How do containers in the same Docker Compose file communicate with each other?**
Compose automatically creates a shared network, and each service is reachable by its service name as a hostname — so a `web` service connects to a database service using the hostname `db` (matching the service name in the YAML), not `localhost` or a hardcoded IP.

**Q: What's the difference between `CMD` and `ENTRYPOINT`?**
`CMD` provides default arguments that can be fully overridden when running the container (`docker run image other-command`). `ENTRYPOINT` sets the command that always runs — arguments passed at `docker run` are appended to it rather than replacing it. They're often combined: `ENTRYPOINT` fixes the executable, `CMD` supplies default arguments that can still be overridden.
