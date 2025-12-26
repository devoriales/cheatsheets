# Devoriales - Docker Cheatsheet
More comprehensive tutorials at [https://devoriales.com](https://devoriales.com)

> A comprehensive guide to Docker commands and best practices for DevOps engineers

## Table of Contents

- [Installation](#installation)
- [Docker CLI Basics](#docker-cli-basics)
- [Container Management](#container-management)
- [Image Management](#image-management)
- [Dockerfile Reference](#dockerfile-reference)
- [Docker Compose](#docker-compose)
- [Networking](#networking)
- [Volumes & Storage](#volumes--storage)
- [Logs & Debugging](#logs--debugging)
- [System Management](#system-management)
- [Best Practices](#best-practices)

---

## Installation

### macOS (Apple Silicon)

```bash
# Using Homebrew
brew install --cask docker

# Verify installation
docker --version
docker compose version
```

### Linux (Ubuntu/Debian)

```bash
# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add user to docker group (avoid using sudo)
sudo usermod -aG docker $USER
```

---

## Docker CLI Basics

### Getting Help

```bash
# Display help for any command
docker --help
docker run --help
docker build --help
```

### Version Information

```bash
# Check Docker version
docker --version
docker version

# Display system-wide information
docker info
```

### Connect to Remote Docker Daemon

```bash
# Via TCP socket
docker -H tcp://192.168.1.100:2376 ps

# Via SSH
docker -H ssh://user@192.168.64.5 ps
```

---

## Container Management

### Running Containers

```bash
# Run a container (basic)
docker run <image>

# Run in detached mode
docker run -d <image>

# Run with name
docker run --name my-container <image>

# Run with port mapping
docker run -p 8080:80 <image>

# Run with environment variables
docker run -e "ENV_VAR=value" <image>

# Run with volume mount
docker run -v /host/path:/container/path <image>

# Run with resource limits
docker run --memory="512m" --cpus="1.0" <image>

# Run interactively with shell
docker run -it <image> /bin/bash

# Run and remove after exit
docker run --rm <image>

# Complete example
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -e NGINX_HOST=example.com \
  --restart unless-stopped \
  nginx:latest
```

### Container Lifecycle

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List containers with custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# Start a stopped container
docker start <container>

# Stop a running container
docker stop <container>

# Restart a container
docker restart <container>

# Pause a container
docker pause <container>

# Unpause a container
docker unpause <container>

# Kill a container (force stop)
docker kill <container>

# Remove a container
docker rm <container>

# Remove a running container (force)
docker rm -f <container>

# Remove multiple containers
docker rm container1 container2 container3

# Stop and remove a container
docker stop <container> && docker rm <container>
```

### Container Interaction

```bash
# Execute command in running container
docker exec <container> <command>

# Open interactive shell
docker exec -it <container> /bin/bash
docker exec -it <container> sh

# Execute as specific user
docker exec -u root -it <container> /bin/bash

# Copy files to/from container
docker cp /local/path <container>:/container/path
docker cp <container>:/container/path /local/path

# Attach to running container
docker attach <container>

# View container port mappings
docker port <container>

# View container processes
docker top <container>

# View container resource usage
docker stats <container>

# View live container statistics
docker stats

# Inspect container details
docker inspect <container>

# Inspect specific field (using Go template)
docker inspect --format='{{.State.Running}}' <container>
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container>
```

---

## Image Management

### Working with Images

```bash
# List images
docker images
docker image ls

# Search for images on Docker Hub
docker search <term>

# Pull an image
docker pull <image>
docker pull <image>:<tag>

# Pull specific platform (Apple Silicon)
docker pull --platform linux/arm64 <image>

# Push an image to registry
docker push <username>/<image>:<tag>

# Tag an image
docker tag <source-image> <target-image>:<tag>

# Remove an image
docker rmi <image>
docker image rm <image>

# Remove multiple images
docker rmi image1 image2 image3

# Remove dangling images
docker image prune

# Remove all unused images
docker image prune -a

# Inspect image details
docker inspect <image>

# View image history
docker history <image>

# Save image to tar archive
docker save -o image.tar <image>

# Load image from tar archive
docker load -i image.tar

# Export container as tar
docker export <container> > container.tar

# Import tar as image
docker import container.tar <image>:<tag>
```

### Building Images

```bash
# Build from Dockerfile in current directory
docker build -t <image>:<tag> .

# Build with custom Dockerfile
docker build -f Dockerfile.prod -t <image>:<tag> .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t <image>:<tag> .

# Build without cache
docker build --no-cache -t <image>:<tag> .

# Build and push (using buildx)
docker buildx build --push -t <username>/<image>:<tag> .

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t <username>/<image>:<tag> \
  --push .

# Build with target stage (multi-stage builds)
docker build --target production -t <image>:<tag> .

# View build history
docker buildx history <image>

# View build logs
docker buildx history logs <build-id>
```

### Registry Operations

```bash
# Login to Docker Hub
docker login

# Login to private registry
docker login registry.example.com

# Logout
docker logout

# Tag for private registry
docker tag <image> registry.example.com/<image>:<tag>

# Push to private registry
docker push registry.example.com/<image>:<tag>
```

---

## Dockerfile Reference

### Basic Structure

```dockerfile
# Base image
FROM node:18-alpine

# Metadata
LABEL maintainer="devops@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY package*.json ./
COPY . .

# Install dependencies
RUN npm install --production

# Set environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Expose port
EXPOSE 3000

# Create user and switch
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js

# Default command
CMD ["node", "server.js"]
```

### Multi-Stage Build Example

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /myapp ./cmd

# Production stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /myapp .
EXPOSE 8080
CMD ["./myapp"]
```

### Dockerfile Instructions

```dockerfile
# FROM - Base image
FROM ubuntu:22.04
FROM node:18-alpine AS builder

# RUN - Execute commands during build
RUN apt-get update && apt-get install -y curl
RUN npm install --production

# CMD - Default command (can be overridden)
CMD ["executable", "param1", "param2"]
CMD command param1 param2

# ENTRYPOINT - Fixed command (args can be passed)
ENTRYPOINT ["executable"]
ENTRYPOINT exec top -b

# COPY - Copy files from build context
COPY . /app
COPY --from=builder /app/dist ./dist

# ADD - Copy with additional features (tar auto-extraction)
ADD archive.tar.gz /app

# WORKDIR - Set working directory
WORKDIR /app

# ENV - Set environment variables
ENV NODE_ENV=production
ENV API_KEY=default_key

# ARG - Build-time variables
ARG VERSION=latest
ARG BUILD_DATE

# EXPOSE - Document port usage
EXPOSE 80 443

# VOLUME - Define mount point
VOLUME ["/data"]

# USER - Switch user
USER nodejs

# LABEL - Add metadata
LABEL version="1.0"
LABEL description="My application"

# HEALTHCHECK - Container health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# SHELL - Change default shell
SHELL ["/bin/bash", "-c"]

# ONBUILD - Trigger for child images
ONBUILD COPY . /app
```

### Best Practices in Dockerfile

```dockerfile
# Use specific tags, not 'latest'
FROM node:18.17-alpine

# Combine RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*

# Use .dockerignore file
# .dockerignore content:
# node_modules
# .git
# .env
# *.md

# Order commands by change frequency (cache optimization)
COPY package*.json ./
RUN npm install
COPY . .

# Use bind mounts for temporary files
RUN --mount=type=bind,source=requirements.txt,target=/tmp/requirements.txt \
    pip install --requirement /tmp/requirements.txt

# Run as non-root user
USER node

# Use COPY instead of ADD (unless you need ADD features)
COPY --chown=node:node . .
```

---

## Docker Compose

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    depends_on:
      - db
    volumes:
      - ./src:/app/src
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Docker Compose Commands

```bash
# Start services in detached mode
docker compose up -d

# Start specific services
docker compose up -d web db

# Build and start
docker compose up --build

# View logs
docker compose logs
docker compose logs -f web
docker compose logs --tail=100 web

# Stop services
docker compose stop

# Stop specific service
docker compose stop web

# Start stopped services
docker compose start

# Restart services
docker compose restart

# Pause services
docker compose pause

# Unpause services
docker compose unpause

# Stop and remove containers, networks
docker compose down

# Stop and remove, including volumes
docker compose down -v

# Remove orphaned containers
docker compose down --remove-orphans

# List running services
docker compose ps

# Execute command in service
docker compose exec web sh

# Run one-off command
docker compose run web npm test

# View service logs
docker compose logs web

# Scale services
docker compose up -d --scale web=3

# Validate compose file
docker compose config

# View service configuration
docker compose config --services

# Pull service images
docker compose pull

# Build service images
docker compose build

# Push service images
docker compose push
```

### Advanced Compose Features

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        - BUILD_VERSION=1.0.0
      cache_from:
        - myapp:latest
    
    # Resource limits
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Profiles (conditional startup)
    profiles:
      - production
    
    # Logging
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### Using Profiles

```bash
# Start services with specific profile
docker compose --profile debug up

# Stop services with profile
docker compose --profile debug down

# Multiple profiles
docker compose --profile frontend --profile backend up
```

---

## Networking

### Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Create bridge network
docker network create -d bridge my-bridge-network

# Create network with subnet
docker network create --subnet=172.18.0.0/16 my-custom-network

# Inspect network
docker network inspect my-network

# Connect container to network
docker network connect my-network my-container

# Disconnect container from network
docker network disconnect my-network my-container

# Remove network
docker network rm my-network

# Remove unused networks
docker network prune
```

### Container Networking

```bash
# Run container with specific network
docker run -d --network my-network --name web nginx

# Run with network alias
docker run -d --network my-network --network-alias db postgres

# Run with host networking
docker run -d --network host nginx

# Run without networking
docker run -d --network none nginx

# Connect to multiple networks
docker network connect network1 container1
docker network connect network2 container1
```

### Network Types

| Network Type | Description | Use Case |
|--------------|-------------|----------|
| bridge | Default network, isolated | Development, single host |
| host | Share host's network | Performance critical |
| overlay | Multi-host networking | Docker Swarm, multi-host |
| macvlan | Assign MAC address | Legacy app requirements |
| none | No networking | Security isolation |

---

## Volumes & Storage

### Volume Management

```bash
# Create volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove unused volumes
docker volume prune

# Remove all unused volumes
docker volume prune -a
```

### Using Volumes

```bash
# Named volume
docker run -d -v my-volume:/data nginx

# Bind mount (absolute path)
docker run -d -v /host/path:/container/path nginx

# Bind mount (relative path)
docker run -d -v $(pwd):/app node:18

# Read-only bind mount
docker run -d -v $(pwd):/app:ro nginx

# tmpfs mount (memory only)
docker run -d --tmpfs /tmp nginx

# Volume with specific driver
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  my-nfs-volume
```

### Volume Backup and Restore

```bash
# Backup volume to tar
docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz -C /data .

# Restore volume from tar
docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/backup.tar.gz"
```

---

## Logs & Debugging

### Viewing Logs

```bash
# View container logs
docker logs <container>

# Follow logs (like tail -f)
docker logs -f <container>

# View last N lines
docker logs --tail 100 <container>

# View logs with timestamps
docker logs -t <container>

# View logs since specific time
docker logs --since 2h <container>
docker logs --since 2023-01-01T10:00:00 <container>

# View logs until specific time
docker logs --until 2h <container>

# View daemon logs (Linux with systemd)
journalctl -xu docker.service

# View container log file path
docker inspect --format='{{.LogPath}}' <container>
```

### Debugging Containers

```bash
# Inspect container details
docker inspect <container>

# View container processes
docker top <container>

# View container stats
docker stats <container>

# View container changes
docker diff <container>

# View container events
docker events

# View container resource usage
docker stats --no-stream

# Check container health
docker inspect --format='{{.State.Health.Status}}' <container>

# View container IP address
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container>

# View container environment variables
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' <container>

# Debug networking issues
docker exec <container> ping <other-container>
docker exec <container> nslookup <hostname>
docker exec <container> curl http://service:port
```

### Logging Drivers

```bash
# Configure logging driver for container
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx

# Available log drivers:
# - json-file (default)
# - syslog
# - journald
# - gelf
# - fluentd
# - awslogs
# - splunk
# - local

# Check container's logging driver
docker inspect -f '{{.HostConfig.LogConfig.Type}}' <container>
```

---

## System Management

### System Information

```bash
# Display system information
docker info

# Display Docker version
docker version

# Show Docker disk usage
docker system df

# Show detailed usage
docker system df -v
```

### Cleaning Up

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything (containers, images, networks)
docker system prune

# Remove everything including volumes
docker system prune --volumes

# Remove everything without confirmation
docker system prune -af --volumes

# Remove all stopped containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Stop and remove all containers
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

### Resource Management

```bash
# Set resource limits
docker run -d \
  --memory="512m" \
  --memory-swap="1g" \
  --cpus="1.5" \
  --pids-limit=100 \
  nginx

# Update container resources
docker update --memory="1g" --cpus="2" <container>

# View container resource usage
docker stats

# Limit container restarts
docker run -d --restart=on-failure:5 nginx
```

### Context Management

```bash
# List contexts
docker context ls

# Create context for remote daemon
docker context create remote-docker \
  --docker "host=ssh://user@remote-host"

# Switch context
docker context use remote-docker

# Remove context
docker context rm remote-docker

# Show current context
docker context show
```

---

## Best Practices

### Security

```bash
# Run as non-root user
docker run -u 1000:1000 <image>

# Read-only root filesystem
docker run --read-only <image>

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE <image>

# No new privileges
docker run --security-opt=no-new-privileges <image>

# Use secrets for sensitive data
echo "my-secret" | docker secret create my-secret -
docker service create --secret my-secret <image>
```

### Image Optimization

```dockerfile
# Use multi-stage builds
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]

# Minimize layers
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*

# Use .dockerignore
# .dockerignore:
node_modules
.git
.env
*.md
.DS_Store
```

### Performance Tips

```bash
# Use BuildKit for faster builds
export DOCKER_BUILDKIT=1
docker build .

# Use build cache
docker build --cache-from myapp:latest -t myapp:new .

# Parallel builds with docker compose
docker compose build --parallel

# Use volume for better I/O performance
docker run -v my-data:/data <image>

# Use tmpfs for temporary data
docker run --tmpfs /tmp:rw,noexec,nosuid,size=100m <image>
```

### CI/CD Integration

```yaml
# GitHub Actions example
name: Docker Build and Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: user/app:latest
          cache-from: type=registry,ref=user/app:latest
          cache-to: type=inline
```

### Kubernetes Integration

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
```

---

## Quick Reference

### Common Flags

| Flag | Description |
|------|-------------|
| `-d` | Detached mode (run in background) |
| `-it` | Interactive with TTY |
| `-p` | Publish port (HOST:CONTAINER) |
| `-v` | Mount volume |
| `-e` | Set environment variable |
| `--name` | Assign container name |
| `--rm` | Remove container after exit |
| `--network` | Connect to network |
| `-f` | Force operation |
| `--build` | Build before starting |

### Container States

```bash
# created → running → stopped → removed
#                  ↓
#               paused
```

### Port Mapping Examples

```bash
# Single port
-p 8080:80

# Multiple ports
-p 8080:80 -p 8443:443

# Bind to specific interface
-p 127.0.0.1:8080:80

# Random host port
-p 80

# UDP port
-p 53:53/udp

# Range of ports
-p 8000-9000:8000-9000
```

---

## Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Labs](https://github.com/docker/labs)
- [Play with Docker](https://labs.play-with-docker.com/)
- [Devoriales - DevOps Blog](https://devoriales.com)

---

**Author**: Devoriales.com  
**Last Updated**: December 2024  
**Version**: 1.0

---

_For more DevOps cheatsheets and tutorials, visit [devoriales.com](https://devoriales.com)_
