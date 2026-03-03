# 🐳 Docker

> Frequently used Docker commands for containers, images, volumes, and networks.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation](#installation)
- [Images](#images)
- [Containers](#containers)
- [Volumes](#volumes)
- [Networks](#networks)
- [Docker Compose](#docker-compose)
- [Registry & Hub](#registry--hub)
- [System & Cleanup](#system--cleanup)
- [Dockerfile Essentials](#dockerfile-essentials)

---

## Installation

```bash
# Install Docker on Ubuntu
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# Verify installation
docker --version
docker info
```

---

## Images

```bash
# Pull an image
docker pull nginx
docker pull nginx:1.25
docker pull ubuntu:22.04

# List images
docker images
docker image ls

# Build an image from Dockerfile
docker build -t myapp:latest .
docker build -t myapp:v1.0 -f Dockerfile.prod .

# Tag an image
docker tag myapp:latest myrepo/myapp:latest

# Remove an image
docker rmi nginx
docker rmi -f nginx            # force remove
docker image rm nginx:1.25

# Inspect an image
docker inspect nginx
docker history nginx

# Save / load images
docker save -o myapp.tar myapp:latest
docker load -i myapp.tar
```

---

## Containers

```bash
# Run a container
docker run nginx
docker run -d nginx                          # detached
docker run -it ubuntu bash                   # interactive
docker run --name mycontainer -d nginx       # named
docker run -p 8080:80 -d nginx               # port mapping  host:container
docker run -e ENV_VAR=value -d myapp         # environment variable
docker run --rm -it ubuntu bash              # auto-remove on exit
docker run -v /host/path:/container/path -d nginx  # bind mount

# List containers
docker ps                    # running
docker ps -a                 # all (including stopped)

# Start / stop / restart
docker start mycontainer
docker stop mycontainer
docker restart mycontainer
docker pause mycontainer
docker unpause mycontainer

# Remove containers
docker rm mycontainer
docker rm -f mycontainer     # force remove running container
docker rm $(docker ps -aq)   # remove all stopped containers

# Execute command inside container
docker exec -it mycontainer bash
docker exec mycontainer ls /app

# Attach to running container
docker attach mycontainer

# View logs
docker logs mycontainer
docker logs -f mycontainer          # follow
docker logs --tail 100 mycontainer  # last 100 lines

# Copy files
docker cp mycontainer:/app/file.txt ./file.txt
docker cp ./file.txt mycontainer:/app/

# Inspect container
docker inspect mycontainer

# Container stats
docker stats
docker stats mycontainer

# Container processes
docker top mycontainer
```

---

## Volumes

```bash
# Create a volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect myvolume

# Use a volume with a container
docker run -d -v myvolume:/app/data nginx

# Remove a volume
docker volume rm myvolume

# Remove all unused volumes
docker volume prune
```

---

## Networks

```bash
# List networks
docker network ls

# Create a network
docker network create mynetwork
docker network create --driver bridge mynetwork
docker network create --subnet 172.20.0.0/16 mynetwork

# Connect container to network
docker network connect mynetwork mycontainer

# Disconnect container from network
docker network disconnect mynetwork mycontainer

# Inspect network
docker network inspect mynetwork

# Run container on specific network
docker run -d --network mynetwork nginx

# Remove network
docker network rm mynetwork

# Remove all unused networks
docker network prune
```

---

## Docker Compose

```bash
# Start services
docker compose up
docker compose up -d          # detached
docker compose up --build     # rebuild images

# Stop services
docker compose down
docker compose down -v        # also remove volumes
docker compose stop

# View logs
docker compose logs
docker compose logs -f
docker compose logs web       # specific service

# Scale a service
docker compose up --scale web=3

# List services
docker compose ps

# Execute in a service
docker compose exec web bash

# Build images only
docker compose build

# Pull images
docker compose pull

# Remove containers and images
docker compose down --rmi all
```

**Example `docker-compose.yml`:**

```yaml
version: '3.9'
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  pgdata:
```

---

## Registry & Hub

```bash
# Login to Docker Hub
docker login
docker login -u username -p password

# Login to private registry
docker login registry.example.com

# Push an image
docker push myrepo/myapp:latest

# Pull from private registry
docker pull registry.example.com/myapp:latest

# Logout
docker logout
```

---

## System & Cleanup

```bash
# Show disk usage
docker system df

# Remove all unused resources
docker system prune
docker system prune -a        # also remove unused images
docker system prune -a -f     # skip confirmation

# Remove dangling images
docker image prune

# Remove stopped containers
docker container prune

# Remove unused networks
docker network prune

# Remove unused volumes
docker volume prune
```

---

## Dockerfile Essentials

```dockerfile
# Base image
FROM ubuntu:22.04

# Metadata
LABEL maintainer="devops@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY . .
COPY requirements.txt /app/

# Run commands during build
RUN apt-get update && apt-get install -y python3 pip \
    && rm -rf /var/lib/apt/lists/*

# Install dependencies
RUN pip install -r requirements.txt

# Set environment variable
ENV APP_ENV=production
ENV PORT=8080

# Expose port
EXPOSE 8080

# Create volume mount point
VOLUME ["/app/data"]

# Switch user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1

# Default command
CMD ["python3", "app.py"]

# Or use ENTRYPOINT
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

---

[← Back to Home](../README.md)
