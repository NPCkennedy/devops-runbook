# Docker Commands

**Docs:** [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/)

---

## What is Docker?

Docker packages your application and all its dependencies into a container image — a single portable unit that runs the same way everywhere: local machine, CI/CD pipeline, Azure Container Apps.

```
Your code + dependencies + runtime
→ packaged into a Docker image
→ pushed to ACR
→ pulled and run by Container Apps
```

---

## Build an Image

**What:** Builds a Docker image from a Dockerfile.
**When:** Testing your container locally before pushing to ACR.
**Docs:** [docker build](https://docs.docker.com/engine/reference/commandline/build/)

```bash
docker build -t <image-name>:<tag> <path-to-dockerfile-dir>
```

| Flag | Required | Description |
|------|----------|-------------|
| -t | Yes | Name and tag for the image |
| path | Yes | Directory containing the Dockerfile (. = current dir) |

**Example:**
```bash
docker build -t zen-backend:local ./src/backend
```

---

## Run a Container Locally

**What:** Starts a container from an image on your local machine.
**When:** Testing before pushing to Azure.
**Docs:** [docker run](https://docs.docker.com/engine/reference/commandline/run/)

```bash
docker run -d \
  -p <host-port>:<container-port> \
  --name <container-name> \
  -e KEY=VALUE \
  <image-name>:<tag>
```

| Flag | Required | Description |
|------|----------|-------------|
| -d | No | Detached mode — runs in background |
| -p | No | Port mapping: host:container |
| --name | No | Name for the container |
| -e | No | Set environment variable |

**Example:**
```bash
docker run -d -p 8000:8000 --name backend zen-backend:local
```

---

## List Running Containers

**What:** Shows all currently running containers.
**Docs:** [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)

```bash
docker ps
```

All containers including stopped:
```bash
docker ps -a
```

---

## View Logs

**What:** Shows stdout/stderr from a running container.
**When:** Debugging a container that is failing or behaving unexpectedly.
**Docs:** [docker logs](https://docs.docker.com/engine/reference/commandline/logs/)

```bash
docker logs <container-name>
```

Follow live logs:
```bash
docker logs -f <container-name>
```

---

## Stop and Remove

**What:** Stops and removes a container.
**Docs:** [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) | [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)

```bash
docker stop <container-name>
docker rm <container-name>
```

Stop and remove in one command:
```bash
docker rm -f <container-name>
```

---

## Tag an Image

**What:** Adds a new tag to an existing image.
**When:** Before pushing to ACR — the image must be tagged with the full ACR login server path.
**Docs:** [docker tag](https://docs.docker.com/engine/reference/commandline/tag/)

```bash
docker tag <source-image>:<tag> <acr-login-server>/<repo>:<tag>
```

**Example:**
```bash
docker tag zen-backend:local zenecr2026.azurecr.io/backend:sha-08643f
```

---

## Push to ACR

**What:** Uploads a tagged image to Azure Container Registry.
**When:** After building and tagging locally — makes the image available for Container Apps.
**Docs:** [docker push](https://docs.docker.com/engine/reference/commandline/push/)

```bash
# Login first
az acr login --name <acr-name>

# Push
docker push <acr-login-server>/<repo>:<tag>
```

**Example:**
```bash
az acr login --name zenecr2026
docker push zenecr2026.azurecr.io/backend:sha-08643f
```

---

## Docker Compose

**What:** Runs multiple containers together defined in a `docker-compose.yml` file.
**When:** Local development — starts all services (frontend, backend, database) together.
**Docs:** [Docker Compose](https://docs.docker.com/compose/reference/)

```bash
# Start all services
docker compose up -d

# Start and rebuild images
docker compose up --build -d

# Stop all services
docker compose down

# View logs for all services
docker compose logs -f

# View logs for one service
docker compose logs -f backend
```

---

## Clean Up

**What:** Removes all stopped containers, unused images, and build cache.
**When:** Freeing up disk space on your local machine.

```bash
docker system prune -a
```

**Warning:** This removes all unused images including ones you may want to keep locally.
