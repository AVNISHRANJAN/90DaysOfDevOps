# 🐳 Docker Complete Command Reference

> A comprehensive README covering every major Docker command with syntax and examples.

---

## 📋 Table of Contents

- [Docker Basics](#-docker-basics)
- [Docker Images](#-docker-images)
- [Docker Containers](#-docker-containers)
- [Docker Volumes](#-docker-volumes)
- [Docker Networks](#-docker-networks)
- [Dockerfile Instructions](#-dockerfile-instructions)
- [Docker Compose](#-docker-compose)
- [System & Cleanup](#-system--cleanup)
- [Registry & Docker Hub](#-registry--docker-hub)
- [Quick Tips & One-Liners](#-quick-tips--one-liners)

---

## ⚙️ Docker Basics

### `docker --version`
Show the installed Docker version.
```bash
docker --version
# Docker version 24.0.5, build ced0996
```

### `docker info`
Display system-wide information including containers, images, storage driver, and runtime details.
```bash
docker info
```

### `docker help`
List all available Docker commands or get help for a specific command.
```bash
docker help
docker COMMAND --help
docker run --help
```

### `docker login`
Log in to Docker Hub or a private registry.
```bash
docker login                            # Docker Hub (prompts for credentials)
docker login -u myuser                  # Specify username
docker login registry.example.com       # Private registry
docker login -u user -p pass registry   # Non-interactive (not recommended)
```

### `docker logout`
Log out from a Docker registry.
```bash
docker logout                           # Docker Hub
docker logout registry.example.com     # Private registry
```

### `docker search`
Search Docker Hub for available images.
```bash
docker search nginx
docker search --filter=stars=100 python     # Filter by star count
docker search --filter=is-official=true ubuntu
docker search --limit 10 node
```

---

## 🖼️ Docker Images

### `docker pull`
Download an image from a registry.
```bash
docker pull nginx                       # Latest tag
docker pull ubuntu:22.04                # Specific tag
docker pull python:3.11-slim            # Slim variant
docker pull registry.example.com/myapp:prod  # Private registry
```

### `docker push`
Upload a local image to a registry.
```bash
docker push myuser/myapp:latest
docker push registry.example.com/myapp:1.0
```

### `docker images`
List all locally available images.
```bash
docker images                           # List all images
docker images -a                        # Include intermediate images
docker images -q                        # Only show image IDs
docker images --digests                 # Show image digests
docker images --filter dangling=true    # Show dangling images
docker images ubuntu                    # Filter by name
```

### `docker build`
Build an image from a Dockerfile.
```bash
docker build -t myapp:1.0 .                         # Build from current directory
docker build -t myapp:latest -f Dockerfile.prod .   # Custom Dockerfile
docker build --no-cache -t myapp .                  # Ignore build cache
docker build --build-arg VERSION=2.0 -t myapp .     # Pass build arguments
docker build --target builder -t myapp .            # Multi-stage: specific stage
docker build -t myapp https://github.com/user/repo  # Build from URL
```

### `docker tag`
Create a new tag (alias) for an existing image.
```bash
docker tag myapp:latest myuser/myapp:1.0
docker tag myapp registry.example.com/myapp:prod
```

### `docker rmi`
Remove one or more images.
```bash
docker rmi nginx                        # Remove by name
docker rmi nginx:1.25                   # Remove specific tag
docker rmi -f nginx                     # Force remove
docker rmi $(docker images -q)          # Remove all images
docker rmi $(docker images -f dangling=true -q)  # Remove dangling images
```

### `docker inspect` (Image)
Return detailed low-level JSON information about an image.
```bash
docker inspect ubuntu:22.04
docker inspect --format='{{.Id}}' ubuntu:22.04       # Format output
docker inspect --format='{{.Config.Env}}' ubuntu     # Show env vars
```

### `docker history`
Show the build history (layers) of an image.
```bash
docker history nginx
docker history --no-trunc nginx         # Show full commands
```

### `docker save`
Export one or more images to a tar archive.
```bash
docker save -o nginx.tar nginx:latest
docker save nginx ubuntu | gzip > images.tar.gz     # Multiple images
```

### `docker load`
Load an image from a tar archive.
```bash
docker load -i nginx.tar
docker load < nginx.tar
cat images.tar.gz | docker load
```

### `docker import`
Create an image from a tarball (filesystem snapshot).
```bash
docker import container.tar myimage:imported
docker import https://example.com/image.tar myimage
```

### `docker commit`
Create a new image from a container's changes.
```bash
docker commit mycontainer myimage:v2
docker commit -m "added curl" -a "Author Name" mycontainer myimage:v2
```

---

## 📦 Docker Containers

### `docker run`
Create and start a new container from an image.
```bash
# Basic run
docker run nginx

# Detached (background) mode
docker run -d nginx

# Interactive terminal
docker run -it ubuntu:22.04 bash

# Named container
docker run -d --name web nginx

# Port mapping  HOST:CONTAINER
docker run -d -p 8080:80 nginx
docker run -d -p 127.0.0.1:8080:80 nginx   # Bind to specific host IP

# Environment variables
docker run -e NODE_ENV=production myapp
docker run --env-file .env myapp

# Volume mount
docker run -v /host/path:/container/path nginx
docker run -v myvolume:/data nginx

# Auto-remove on exit
docker run --rm alpine echo "hello world"

# Resource limits
docker run --memory=256m --cpus=0.5 myapp
docker run --memory=512m --memory-swap=1g myapp

# Restart policy
docker run -d --restart=always nginx
docker run -d --restart=on-failure:3 myapp

# Network
docker run --network mynet nginx
docker run --network=host nginx
docker run --network=none alpine

# Hostname & DNS
docker run --hostname myhost --dns 8.8.8.8 nginx

# User
docker run --user 1001:1001 myapp
docker run -u node myapp

# Working directory
docker run -w /app myapp

# Read-only filesystem
docker run --read-only nginx

# Privileged mode
docker run --privileged myapp
```

### `docker ps`
List containers.
```bash
docker ps                               # Running containers only
docker ps -a                            # All containers (including stopped)
docker ps -q                            # Only container IDs
docker ps -l                            # Latest created container
docker ps -n 5                          # Last 5 containers
docker ps --filter status=exited        # Filter by status
docker ps --filter name=web             # Filter by name
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

### `docker start`
Start one or more stopped containers.
```bash
docker start web
docker start web db cache               # Start multiple
docker start -a web                     # Attach stdout/stderr
docker start -i web                     # Attach and connect stdin
```

### `docker stop`
Stop a running container gracefully (sends SIGTERM, then SIGKILL).
```bash
docker stop web
docker stop -t 5 web                    # Wait 5 seconds before killing
docker stop $(docker ps -q)             # Stop all running containers
```

### `docker restart`
Stop and start a container.
```bash
docker restart web
docker restart -t 10 web               # 10s timeout before kill
```

### `docker pause` / `docker unpause`
Suspend / resume all processes in a container.
```bash
docker pause web
docker unpause web
```

### `docker kill`
Send a signal to a container (default: SIGKILL).
```bash
docker kill web                         # SIGKILL
docker kill -s SIGHUP web              # Custom signal
docker kill -s SIGTERM web
```

### `docker rm`
Remove one or more stopped containers.
```bash
docker rm web
docker rm -f web                        # Force remove running container
docker rm -v web                        # Also remove associated volumes
docker rm $(docker ps -aq)              # Remove all stopped containers
docker rm $(docker ps -aq -f status=exited)
```

### `docker exec`
Run a command inside a running container.
```bash
docker exec -it web bash                # Interactive shell
docker exec -it web sh                  # For Alpine-based images
docker exec web ls /app                 # Non-interactive command
docker exec -e VAR=value web env        # With env variable
docker exec -u root web bash            # As specific user
docker exec -w /tmp web ls              # Different working dir
```

### `docker attach`
Attach your terminal to a running container's stdin/stdout/stderr.
```bash
docker attach web
docker attach --no-stdin web            # Read-only attach
```
> **Note:** Use `Ctrl+P` then `Ctrl+Q` to detach without stopping the container.

### `docker logs`
Fetch the logs of a container.
```bash
docker logs web
docker logs -f web                      # Follow (stream) logs
docker logs --tail 100 web              # Last 100 lines
docker logs --since 30m web             # Logs from last 30 minutes
docker logs --since 2024-01-01 web      # Since a date
docker logs --until 2024-06-01 web      # Until a date
docker logs -t web                      # Show timestamps
```

### `docker inspect` (Container)
Return detailed JSON info about a container.
```bash
docker inspect web
docker inspect --format='{{.State.Status}}' web
docker inspect --format='{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web
docker inspect --format='{{.Mounts}}' web
```

### `docker stats`
Display a live stream of container resource usage.
```bash
docker stats                            # All running containers
docker stats web db                     # Specific containers
docker stats --no-stream web            # Single snapshot
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### `docker top`
Display running processes inside a container.
```bash
docker top web
docker top web aux                      # Pass ps options
```

### `docker diff`
Show filesystem changes made to a container since it started.
```bash
docker diff web
# A = Added, C = Changed, D = Deleted
```

### `docker cp`
Copy files/folders between a container and the local filesystem.
```bash
docker cp web:/app/log.txt .                    # Container → Host
docker cp ./config.json web:/etc/app/           # Host → Container
docker cp web:/var/log/nginx/error.log ./logs/
```

### `docker rename`
Rename a container.
```bash
docker rename old_name new_name
docker rename web frontend
```

### `docker update`
Update resource limits of a running container.
```bash
docker update --memory 512m web
docker update --cpus 1.5 web
docker update --restart=always web
docker update --memory 256m --cpus 0.5 web db   # Multiple containers
```

### `docker wait`
Block until a container stops, then print its exit code.
```bash
docker wait web
```

### `docker port`
List port mappings for a container.
```bash
docker port web                         # All mappings
docker port web 80                      # Specific container port
docker port web 80/tcp
```

### `docker export`
Export a container's filesystem as a tar archive.
```bash
docker export web > web_backup.tar
docker export -o web_backup.tar web
```

---

## 💾 Docker Volumes

### `docker volume create`
Create a named volume.
```bash
docker volume create mydata
docker volume create --driver local mydata
docker volume create --opt type=nfs mydata
```

### `docker volume ls`
List all volumes.
```bash
docker volume ls
docker volume ls -q                     # Only volume names
docker volume ls --filter dangling=true
```

### `docker volume inspect`
Display detailed information about a volume.
```bash
docker volume inspect mydata
docker volume inspect --format '{{.Mountpoint}}' mydata
```

### `docker volume rm`
Remove one or more volumes.
```bash
docker volume rm mydata
docker volume rm vol1 vol2 vol3         # Multiple volumes
```

### `docker volume prune`
Remove all unused local volumes.
```bash
docker volume prune
docker volume prune -f                  # Skip confirmation
docker volume prune --filter label=env=dev
```

### Volume Mount Types in `docker run`

**Named Volume:**
```bash
docker run -d -v mydata:/var/lib/mysql mysql
docker run -d --mount type=volume,src=mydata,dst=/var/lib/mysql mysql
```

**Bind Mount (host directory):**
```bash
docker run -d -v /host/path:/container/path nginx
docker run -d --mount type=bind,src=$(pwd),dst=/app node
docker run -d --mount type=bind,src=$(pwd),dst=/app,readonly node  # Read-only
```

**tmpfs Mount (in-memory):**
```bash
docker run --mount type=tmpfs,dst=/tmp nginx
docker run --mount type=tmpfs,dst=/tmp,tmpfs-size=64m nginx
```

---

## 🌐 Docker Networks

### `docker network create`
Create a new network.
```bash
docker network create mynet
docker network create --driver bridge mynet         # Bridge (default)
docker network create --driver overlay mynet        # Overlay (Swarm)
docker network create --driver host mynet           # Host
docker network create --subnet=192.168.1.0/24 mynet
docker network create --subnet=192.168.1.0/24 --gateway=192.168.1.1 mynet
docker network create --internal mynet              # No external access
```

### `docker network ls`
List all networks.
```bash
docker network ls
docker network ls -q                                # Only IDs
docker network ls --filter driver=bridge
docker network ls --filter name=my
```

### `docker network inspect`
Show detailed information about a network.
```bash
docker network inspect bridge
docker network inspect mynet
docker network inspect --format='{{range .Containers}}{{.Name}} {{end}}' mynet
```

### `docker network connect`
Connect a running container to a network.
```bash
docker network connect mynet web
docker network connect --alias db mynet mysql       # With alias
docker network connect --ip 192.168.1.10 mynet web  # Static IP
```

### `docker network disconnect`
Disconnect a container from a network.
```bash
docker network disconnect mynet web
docker network disconnect -f mynet web              # Force
```

### `docker network rm`
Remove one or more networks.
```bash
docker network rm mynet
docker network rm net1 net2 net3
```

### `docker network prune`
Remove all unused networks.
```bash
docker network prune
docker network prune -f                             # Skip confirmation
```

### Network Modes in `docker run`
```bash
docker run --network=bridge nginx       # Default bridge (default)
docker run --network=host nginx         # Share host network stack
docker run --network=none alpine        # No network access
docker run --network=mynet nginx        # Custom named network
docker run --network=container:web nginx  # Share another container's network
```

---

## 📝 Dockerfile Instructions

```dockerfile
# ── Base Image ───────────────────────────────────────
FROM ubuntu:22.04
FROM python:3.11-slim AS builder         # Named stage for multi-stage

# ── Metadata ─────────────────────────────────────────
LABEL maintainer="dev@example.com"
LABEL version="1.0" description="My App"

# ── Build Arguments (build-time only) ────────────────
ARG VERSION=1.0
ARG APP_ENV=production

# ── Environment Variables ─────────────────────────────
ENV NODE_ENV=production
ENV PORT=3000 HOST=0.0.0.0

# ── Working Directory ────────────────────────────────
WORKDIR /app

# ── Copy Files ───────────────────────────────────────
COPY . .                                 # Copy everything
COPY package*.json ./                    # Specific files
COPY --chown=node:node . /app            # With ownership
COPY --from=builder /app/dist ./dist     # From another stage

# ── ADD (like COPY + URL fetch + tar extraction) ──────
ADD https://example.com/file.tar.gz /opt/
ADD archive.tar.gz /opt/                 # Auto-extracts tar

# ── Run Commands ─────────────────────────────────────
RUN apt-get update && apt-get install -y curl git \
    && rm -rf /var/lib/apt/lists/*
RUN pip install -r requirements.txt

# ── User ─────────────────────────────────────────────
USER node
USER 1001:1001

# ── Expose Port ──────────────────────────────────────
EXPOSE 80
EXPOSE 3000/tcp
EXPOSE 8080 9090

# ── Volume Mount Point ───────────────────────────────
VOLUME ["/var/lib/mysql"]
VOLUME /data /logs

# ── Health Check ─────────────────────────────────────
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost/ || exit 1
HEALTHCHECK NONE                         # Disable inherited healthcheck

# ── Default Command (overridable) ────────────────────
CMD ["nginx", "-g", "daemon off;"]
CMD ["python", "app.py"]

# ── Entrypoint (fixed executable) ────────────────────
ENTRYPOINT ["python"]
CMD ["app.py"]                           # Default arg to ENTRYPOINT

# ── Shell Form vs Exec Form ──────────────────────────
RUN apt-get install -y curl              # Shell form (uses /bin/sh -c)
RUN ["apt-get", "install", "-y", "curl"] # Exec form (no shell)

# ── ONBUILD (trigger when used as base) ──────────────
ONBUILD COPY . /app
ONBUILD RUN npm install

# ── STOPSIGNAL ───────────────────────────────────────
STOPSIGNAL SIGTERM

# ── SHELL ────────────────────────────────────────────
SHELL ["/bin/bash", "-c"]
```

### Multi-Stage Build Example
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🔧 Docker Compose

### `docker compose up`
Build, create, and start services.
```bash
docker compose up                       # Foreground
docker compose up -d                    # Detached (background)
docker compose up --build               # Rebuild images before starting
docker compose up -d --build            # Detached + rebuild
docker compose up web                   # Only a specific service
docker compose up --scale web=3         # Scale a service
docker compose up --force-recreate      # Recreate containers even if unchanged
docker compose up --remove-orphans      # Remove orphan containers
```

### `docker compose down`
Stop and remove containers and networks.
```bash
docker compose down
docker compose down -v                  # Also remove volumes
docker compose down --rmi all           # Also remove images
docker compose down --remove-orphans
```

### `docker compose build`
Build or rebuild service images.
```bash
docker compose build
docker compose build web                # Specific service
docker compose build --no-cache         # Ignore cache
docker compose build --pull             # Always pull base images
```

### `docker compose start` / `stop` / `restart`
```bash
docker compose start                    # Start existing containers
docker compose start web db
docker compose stop                     # Stop running containers
docker compose stop web
docker compose restart                  # Restart services
docker compose restart web
```

### `docker compose ps`
List containers for the current Compose project.
```bash
docker compose ps
docker compose ps -a                    # Include stopped
docker compose ps -q                    # Only IDs
docker compose ps web                   # Specific service
```

### `docker compose logs`
View output logs from services.
```bash
docker compose logs
docker compose logs -f                  # Follow logs
docker compose logs web                 # Specific service
docker compose logs -f --tail=100 web
docker compose logs --since 30m
```

### `docker compose exec`
Execute a command in a running service container.
```bash
docker compose exec web bash
docker compose exec web sh
docker compose exec db mysql -u root -p
docker compose exec -u root web bash    # As specific user
docker compose exec -e VAR=val web env
```

### `docker compose run`
Run a one-off command on a service.
```bash
docker compose run web python manage.py migrate
docker compose run --rm web python manage.py createsuperuser
docker compose run -e DEBUG=true web python app.py
docker compose run --no-deps web bash   # Don't start linked services
```

### `docker compose pull` / `push`
```bash
docker compose pull                     # Pull all service images
docker compose pull db                  # Specific service
docker compose push                     # Push all service images
docker compose push web
```

### `docker compose config`
Validate and view the merged Compose configuration.
```bash
docker compose config
docker compose config --services        # List service names
docker compose config --volumes         # List volume names
docker compose config -q                # Validate only (quiet)
```

### `docker compose scale` *(legacy, use --scale instead)*
```bash
docker compose up --scale web=3 --scale worker=2
```

### `docker compose cp`
Copy files between service containers and the host.
```bash
docker compose cp web:/app/log.txt .
docker compose cp ./config.json web:/etc/app/
```

### `docker compose top`
Display running processes in service containers.
```bash
docker compose top
docker compose top web
```

### `docker compose pause` / `unpause`
```bash
docker compose pause
docker compose unpause
docker compose pause web
```

### Sample `compose.yml`
```yaml
version: "3.9"

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: myapp:latest
    container_name: myapp_web
    restart: always
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    env_file:
      - .env
    volumes:
      - ./app:/app
      - static_files:/app/static
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15
    container_name: myapp_db
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
  static_files:

networks:
  frontend:
  backend:
    driver: bridge
```

---

## 🧹 System & Cleanup

### `docker system df`
Show disk usage by Docker objects.
```bash
docker system df
docker system df -v                     # Verbose (per object)
```

### `docker system prune`
Remove all stopped containers, dangling images, unused networks, and build cache.
```bash
docker system prune
docker system prune -f                  # No confirmation prompt
docker system prune -a                  # Also remove unused images
docker system prune -a -f --volumes     # Nuclear option: everything unused
```

### `docker system info`
Display Docker system-wide information.
```bash
docker system info
docker info                             # Alias
```

### `docker system events`
Get real-time events from the Docker daemon.
```bash
docker system events
docker system events --since '2024-01-01'
docker system events --filter event=start
docker system events --filter container=web
docker system events --format '{{.Type}} {{.Action}} {{.Actor.ID}}'
```

### `docker image prune`
Remove dangling or unused images.
```bash
docker image prune                      # Dangling only
docker image prune -a                   # All unused images
docker image prune -f                   # No confirmation
docker image prune --filter until=24h   # Images older than 24h
```

### `docker container prune`
Remove all stopped containers.
```bash
docker container prune
docker container prune -f
docker container prune --filter until=24h
```

### `docker volume prune`
```bash
docker volume prune
docker volume prune -f
docker volume prune --filter label=env=dev
```

### `docker network prune`
```bash
docker network prune
docker network prune -f
```

### `docker builder prune`
Remove build cache.
```bash
docker builder prune
docker builder prune -a                 # Remove all cache
docker builder prune -f                 # No confirmation
docker builder prune --keep-storage 5GB
```

---

## ☁️ Registry & Docker Hub

### Working with Docker Hub
```bash
# Login
docker login -u yourusername

# Pull public image
docker pull nginx:latest

# Tag for push
docker tag myapp:latest yourusername/myapp:1.0

# Push to Docker Hub
docker push yourusername/myapp:1.0
```

### Working with Private Registry
```bash
# Login to private registry
docker login registry.example.com -u user -p password

# Pull from private registry
docker pull registry.example.com/myteam/myapp:prod

# Tag for private registry
docker tag myapp:latest registry.example.com/myteam/myapp:prod

# Push to private registry
docker push registry.example.com/myteam/myapp:prod

# Logout
docker logout registry.example.com
```

### Run a Local Registry
```bash
docker run -d -p 5000:5000 --name registry registry:2

# Tag for local registry
docker tag myapp localhost:5000/myapp:latest

# Push to local registry
docker push localhost:5000/myapp:latest

# Pull from local registry
docker pull localhost:5000/myapp:latest
```

---

## ⚡ Quick Tips & One-Liners

### Container Management
```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker rm $(docker ps -aq)

# Remove all containers (force)
docker rm -f $(docker ps -aq)

# Remove all exited containers
docker rm $(docker ps -aq -f status=exited)

# Follow logs of the latest container
docker logs -f $(docker ps -lq)

# Get container IP address
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' CONTAINER
```

### Image Management
```bash
# Remove all images
docker rmi $(docker images -q)

# Remove all dangling images
docker rmi $(docker images -f dangling=true -q)

# Remove all untagged images
docker image prune -f
```

### Debugging
```bash
# Shell into a running container
docker exec -it CONTAINER bash

# Shell into a container as root
docker exec -it -u root CONTAINER bash

# Inspect container's environment variables
docker exec CONTAINER env

# Check container resource usage (one shot)
docker stats --no-stream CONTAINER

# Watch container logs in real time
docker logs -f --tail 50 CONTAINER

# Check what ports are exposed
docker port CONTAINER
```

### Resource Limits
```bash
# Limit memory and CPU
docker run --memory=256m --cpus=0.5 IMAGE

# Limit with swap
docker run --memory=512m --memory-swap=1g IMAGE

# Reserve CPU shares
docker run --cpu-shares=512 IMAGE
```

### Volume Shortcuts
```bash
# List volumes and their mount points
docker volume inspect $(docker volume ls -q)

# Remove all unused volumes
docker volume prune -f

# Backup a volume to a tar file
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine \
  tar czf /backup/myvolume_backup.tar.gz -C /data .

# Restore a volume from a tar file
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine \
  tar xzf /backup/myvolume_backup.tar.gz -C /data
```

### Useful Format Tricks
```bash
# List containers with custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

# List images with custom format
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Get just running container names
docker ps --format "{{.Names}}"
```

### Multi-Stage Build One-Liner
```bash
# Build only a specific stage
docker build --target builder -t myapp:build .

# Build production stage
docker build --target production -t myapp:prod .
```

---

## 📌 Common Flags Reference

| Flag | Description |
|------|-------------|
| `-d` | Detached / background mode |
| `-i` | Keep STDIN open |
| `-t` | Allocate pseudo-TTY |
| `-it` | Interactive terminal |
| `-p HOST:CONT` | Port mapping |
| `-v HOST:CONT` | Volume / bind mount |
| `-e KEY=VAL` | Set environment variable |
| `--name` | Assign container name |
| `--rm` | Auto-remove on exit |
| `--network` | Connect to network |
| `--restart` | Restart policy |
| `--memory` | Memory limit |
| `--cpus` | CPU limit |
| `-f` | Force (skip prompts) |
| `-a` | All (include stopped) |
| `-q` | Quiet (IDs only) |

---

## 🔁 Restart Policies

| Policy | Description |
|--------|-------------|
| `no` | Never restart (default) |
| `always` | Always restart |
| `on-failure` | Restart on non-zero exit |
| `on-failure:3` | Restart up to 3 times |
| `unless-stopped` | Always restart unless manually stopped |

```bash
docker run -d --restart=always nginx
docker run -d --restart=on-failure:5 myapp
docker update --restart=unless-stopped CONTAINER
```

---

> 🐳 **Docker version covered:** Docker 24+  
> 📅 **Last updated:** 2025  
> 📄 For the full visual cheat sheet, see `Docker_Cheatsheet.pdf`
