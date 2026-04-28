# Practical 4: Docker Storage - Types & Container Linking

## 📋 Overview
Master Docker storage options and learn how to link containers together.

---

## 💾 Part 1: Types of Docker Storage

### 1. Volumes (Recommended)
Volumes are managed by Docker and stored on the host machine.

#### Creating Volumes
```bash
# Create named volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune
```

#### Using Volumes with Containers
```bash
# Mount volume (created if doesn't exist)
docker run -d -v myvolume:/data --name myapp ubuntu

# Mount multiple volumes
docker run -d \
    -v db-data:/var/lib/mysql \
    -v logs:/var/log/mysql \
    --name mysql mysql:latest

# Read-only volume
docker run -d -v myvolume:/data:ro --name myapp ubuntu
```

#### Volume in Docker Compose
```yaml
version: '3.9'

services:
  app:
    image: myapp:latest
    volumes:
      - app-data:/app/data

  database:
    image: postgres:latest
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  app-data:
  db-data:
```

### 2. Bind Mounts
Map host directory to container path.

#### Basic Usage
```bash
# Mount host directory to container
docker run -d -v /host/path:/container/path --name myapp ubuntu

# Windows
docker run -d -v C:\Users\username\data:/app/data --name myapp ubuntu

# Read-only
docker run -d -v /host/path:/container/path:ro --name myapp ubuntu
```

#### Use Cases
```bash
# Development - sync source code
docker run -d \
    -v $(pwd):/app \
    -v /app/node_modules \
    --name dev-server node:latest

# Database backups
docker run -d \
    -v /backups:/backup \
    --name backup-app ubuntu

# Log collection
docker run -d \
    -v /var/log:/container/logs \
    --name log-app ubuntu
```

### 3. tmpfs Mounts
Temporary storage in container memory (no persistence).

```bash
# Create tmpfs mount
docker run -d --tmpfs /tmp:size=100m --name myapp ubuntu

# tmpfs in compose
services:
  app:
    image: ubuntu
    tmpfs:
      - /tmp:size=100m
```

#### Use Cases
```bash
# Temporary cache
docker run -d --tmpfs /cache ubuntu

# Session storage
docker run -d --tmpfs /sessions:size=50m ubuntu

# Sensitive data
docker run -d --tmpfs /secrets:size=10m ubuntu
```

---

## 🔗 Part 2: Linking Containers

### Method 1: Legacy Link (Deprecated)
```bash
# Create database container
docker run -d --name mysql-db mysql:latest

# Link from app container
docker run -d --link mysql-db:db myapp:latest

# In app container, access via 'db' hostname
# Example: mysql -h db -u root
```

### Method 2: Docker Networks (Recommended)

#### Create Custom Network
```bash
# Create bridge network
docker network create mynetwork

# List networks
docker network ls

# Inspect network
docker network inspect mynetwork

# Remove network
docker network rm mynetwork
```

#### Connect Containers
```bash
# Create network
docker network create app-network

# Run database on network
docker run -d --name database \
    --network app-network \
    mysql:latest

# Run app connected to network
docker run -d --name webapp \
    --network app-network \
    -p 8080:8080 \
    myapp:latest

# Connection string in app: mysql://database:3306
# Docker resolves 'database' hostname automatically
```

#### Multi-Network Connection
```bash
# Create two networks
docker network create frontend
docker network create backend

# Create web container
docker run -d --name web \
    --network frontend \
    nginx:latest

# Create app on both networks
docker run -d --name app \
    --network backend \
    myapp:latest

# Connect app to frontend
docker network connect frontend app

# Now app can communicate with both web and database
```

### Method 3: Docker Compose (Automatic Networking)
```yaml
version: '3.9'

services:
  database:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: root123
    networks:
      - app-network

  webapp:
    image: myapp:latest
    ports:
      - "8080:8080"
    depends_on:
      - database
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

```bash
# Start - automatic networking
docker-compose up -d

# App can connect to 'database' hostname
# Connection string: mysql://database:3306
```

---

## 🎯 Part 3: Practical Examples

### Example 1: Persistent Database Storage

```bash
# Create volume for database
docker volume create mysql-data

# Run MySQL with persistent volume
docker run -d \
    --name mysql-server \
    -e MYSQL_ROOT_PASSWORD=root123 \
    -v mysql-data:/var/lib/mysql \
    mysql:8.0

# Stop and remove container
docker stop mysql-server
docker rm mysql-server

# Data persists in volume

# Start new container with same volume
docker run -d \
    --name mysql-server \
    -e MYSQL_ROOT_PASSWORD=root123 \
    -v mysql-data:/var/lib/mysql \
    mysql:8.0

# All data is still there
```

### Example 2: Development Environment with Bind Mount

```bash
# Clone or create project directory
mkdir myapp && cd myapp

# Create app files
echo 'print("Hello from container")' > app.py

# Run with bind mount
docker run -it \
    -v $(pwd):/workspace \
    -w /workspace \
    python:3.9 \
    python app.py

# Edit files on host, changes visible in container
```

### Example 3: Linked Application Stack

```bash
# Create network
docker network create stack-network

# Database
docker run -d \
    --name postgres-db \
    --network stack-network \
    -e POSTGRES_PASSWORD=secret \
    postgres:latest

# Cache
docker run -d \
    --name redis-cache \
    --network stack-network \
    redis:latest

# Web Application
docker run -d \
    --name web-app \
    --network stack-network \
    -p 8080:8080 \
    -e DB_HOST=postgres-db \
    -e CACHE_HOST=redis-cache \
    myapp:latest

# All containers can communicate by hostname
```

### Example 4: Volume Sharing Between Containers

```bash
# Create shared volume
docker volume create shared-data

# Container 1: Write data
docker run -d \
    --name writer \
    -v shared-data:/shared \
    ubuntu \
    bash -c "echo 'Hello' > /shared/message.txt && sleep 3600"

# Container 2: Read data
docker run -it \
    --name reader \
    -v shared-data:/shared \
    ubuntu \
    cat /shared/message.txt

# Output: Hello
```

---

## 📋 Storage Comparison

| Feature | Volumes | Bind Mounts | tmpfs |
|---------|---------|------------|-------|
| Persistence | ✅ Yes | ✅ Yes | ❌ No |
| Performance | Good | Good | Excellent |
| Use Case | Production data | Development | Temporary data |
| Management | Docker managed | Host managed | Container only |
| Ease of Use | Easy | Very easy | Easy |
| Portability | Portable | OS specific | N/A |

---

## 🔍 Inspecting Storage

### Volume Information
```bash
# List all volumes
docker volume ls

# Inspect volume location
docker volume inspect mysql-data
# Output shows mount point on host

# Check volume usage
du -sh /var/lib/docker/volumes/mysql-data/_data

# View volume contents
docker run -v mysql-data:/data ubuntu ls -la /data
```

### Container Storage
```bash
# Show running processes in container
docker top container_name

# Show mounted volumes
docker inspect container_name | grep -A 5 Mounts

# Check container disk usage
docker exec container_name du -sh /
```

---

## 🧪 Hands-On Exercises

### Exercise 1: Create and Persist Data
```bash
# 1. Create volume
docker volume create mydata

# 2. Write data
docker run -it -v mydata:/data ubuntu bash
# Inside: echo "Important data" > /data/file.txt && exit

# 3. Verify persistence
docker run -it -v mydata:/data ubuntu cat /data/file.txt
# Should show: Important data
```

### Exercise 2: Linked Containers
```bash
# 1. Create network
docker network create mynet

# 2. Run database
docker run -d --name db --network mynet ubuntu

# 3. Connect from another container
docker run -it --network mynet ubuntu ping db
# Should work!

# 4. Cleanup
docker stop db
docker network rm mynet
```

### Exercise 3: Development Setup
```bash
# 1. Create project
mkdir myproject && cd myproject
echo 'const express = require("express"); ...' > index.js

# 2. Run with bind mount
docker run -d \
    -v $(pwd):/app \
    -w /app \
    -p 3000:3000 \
    node:16 \
    npm start

# 3. Edit files locally, changes apply in container
```

### Exercise 4: Full Stack with Docker Compose
See Docker Compose practical for complete example.

---

## ✅ Checklist

- [ ] Understand volumes vs bind mounts vs tmpfs
- [ ] Create and manage volumes
- [ ] Mount volumes to containers
- [ ] Link containers using networks
- [ ] Use custom networks for inter-container communication
- [ ] Set up development environment with bind mounts
- [ ] Create persistent database storage
- [ ] Understand volume drivers and plugins
- [ ] Know when to use each storage type

