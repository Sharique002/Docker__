# Practical 8: Docker Networks - Types & Configuration

## 📋 Overview
Master Docker networking for container communication and service discovery.

---

## 🌐 Part 1: Docker Network Basics

### Why Networking?
Containers need to communicate with:
- Other containers
- Host machine
- External networks

### Network Modes in Docker

#### 1. Bridge Network (Default)
- Each container gets its own network interface
- Connected through bridge on host
- Good for single-host multi-container apps

#### 2. Host Network
- Container uses host's network namespace
- No network isolation
- Best performance but least flexible

#### 3. Overlay Network
- Spans multiple Docker hosts
- Used in Docker Swarm
- Enables container communication across nodes

#### 4. Macvlan Network
- Container appears as physical device on network
- Direct access to physical network

#### 5. None Network
- No network access
- Used for isolated containers

---

## 📋 Part 2: Bridge Networks

### Default Bridge
```bash
# List networks
docker network ls

# Docker creates default bridge automatically
# Called 'bridge'

# Run container on default bridge
docker run -d --name web nginx:latest

# Container gets IP from bridge
docker inspect web | grep IPAddress
```

### Custom Bridge Networks
```bash
# Create custom bridge
docker network create mybridge

# List networks
docker network ls

# Inspect network
docker network inspect mybridge

# Remove network
docker network rm mybridge

# Run container on custom bridge
docker run -d --name app --network mybridge nginx:latest

# Connect existing container
docker network connect mybridge web

# Disconnect container
docker network disconnect mybridge web
```

### Bridge Network Advantages
```bash
# Create custom network
docker network create app-network

# Container 1
docker run -d --name db --network app-network postgres:latest

# Container 2
docker run -d --name app --network app-network myapp:latest

# Automatic DNS resolution
docker exec app ping db  # Works!

# Default bridge doesn't support automatic DNS
# Must use --link (deprecated) or specify IP
```

---

## 🔗 Part 3: Host and None Networks

### Host Network
```bash
# Container uses host's network
docker run -d --network host --name web nginx:latest

# Ports directly on host
# No port mapping needed
# Access at http://localhost:80

# Disadvantage: no isolation
```

### None Network
```bash
# No network access
docker run -d --network none --name isolated ubuntu

# Container has only loopback interface
docker exec isolated ip addr
# Only lo interface visible
```

---

## 🌍 Part 4: Overlay Networks (Swarm)

### Create Overlay Network
```bash
# Initialize swarm first
docker swarm init

# Create overlay network
docker network create \
  --driver overlay \
  --scope swarm \
  backend

# Use in service
docker service create \
  --name web \
  --network backend \
  nginx:latest
```

### Multi-Host Communication
```bash
# On Manager Node
docker network create \
  --driver overlay \
  --attachable \
  mynetwork

# On any node
docker run -d --name app --network mynetwork myapp:latest

# On another node
docker run -d --name db --network mynetwork postgres:latest

# Containers can communicate across hosts
docker exec app ping db
```

---

## 📡 Part 5: Advanced Networking

### Port Mapping
```bash
# Single port
docker run -p 8080:80 nginx

# Multiple ports
docker run -p 8080:80 -p 8443:443 nginx

# Bind to specific interface
docker run -p 127.0.0.1:8080:80 nginx

# Random port
docker run -p 8080 nginx

# Range of ports
docker run -p 8080-8090:80 nginx
```

### Service Discovery
```bash
# In custom bridge/overlay networks
# Containers can resolve each other by name

docker network create mynet

docker run -d --name web --network mynet nginx
docker run -d --name app --network mynet myapp

# From app container
docker exec app curl http://web:80
# Works! No need to know IP
```

### Linking Containers (Deprecated)
```bash
# Old way (still works)
docker run -d --name db postgres
docker run -d --link db:database myapp

# Adds /etc/hosts entry
# docker run -d --name app --link db myapp
```

### DNS Configuration
```bash
# Custom DNS servers
docker run -d \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  myapp:latest

# DNS search domains
docker run -d \
  --dns-search example.com \
  myapp:latest
```

---

## 🎯 Part 6: Hands-On Examples

### Example 1: Multi-Container App with Custom Bridge

```bash
# Create network
docker network create web-app

# Start database
docker run -d \
  --name postgres \
  --network web-app \
  -e POSTGRES_PASSWORD=root \
  postgres:latest

# Start Redis
docker run -d \
  --name redis \
  --network web-app \
  redis:latest

# Start web app
docker run -d \
  --name app \
  --network web-app \
  -p 8080:3000 \
  -e DB_HOST=postgres \
  -e CACHE_HOST=redis \
  myapp:latest

# Test connectivity
docker exec app curl http://postgres:5432
docker exec app curl http://redis:6379

# Cleanup
docker network rm web-app
```

### Example 2: Exposed Services

```bash
# Create network
docker network create service-mesh

# Create internal service (no port exposure)
docker run -d \
  --name api \
  --network service-mesh \
  myapi:latest

# Create public gateway
docker run -d \
  --name gateway \
  --network service-mesh \
  -p 8080:80 \
  nginx:latest

# External users access gateway
# Gateway communicates with api internally
```

### Example 3: Multi-Network Service

```bash
# Create networks
docker network create frontend
docker network create backend

# Frontend service
docker run -d \
  --name web \
  --network frontend \
  nginx:latest

# Backend service (connects to both networks)
docker run -d --name api myapi:latest

# Connect api to both networks
docker network connect frontend api
docker network connect backend api

# api can talk to both frontend and backend services
```

---

## 📊 Network Types Comparison

| Feature | Bridge | Host | Overlay | Macvlan |
|---------|--------|------|---------|---------|
| Container Isolation | ✅ | ❌ | ✅ | Partial |
| Multi-Host | ❌ | ❌ | ✅ | ❌ |
| Performance | Good | Best | Fair | Excellent |
| Security | Good | Low | Good | Good |
| Complexity | Low | Low | Medium | High |

---

## 🔍 Network Inspection

### View Network Info
```bash
# List all networks
docker network ls

# Inspect network
docker network inspect mybridge

# View connected containers
docker network inspect mybridge | grep Containers -A 20

# View container network settings
docker inspect container_name | grep -A 20 NetworkSettings
```

### Network Diagnostics
```bash
# Check container connectivity
docker exec container1 ping container2

# Check port accessibility
docker exec app curl http://db:5432

# View network interfaces in container
docker exec container ip addr

# View routing table
docker exec container ip route

# Check open ports
docker exec container netstat -tlnp
```

---

## 🧪 Hands-On Exercise: Complete Network Setup

### Setup Multi-Tier Application

```bash
# Step 1: Create networks
docker network create frontend-net
docker network create backend-net

# Step 2: Create services
# Database (backend only)
docker run -d \
  --name postgres \
  --network backend-net \
  -e POSTGRES_PASSWORD=root \
  postgres:latest

# API Server (both networks)
docker run -d --name api myapi:latest
docker network connect backend-net api
docker network connect frontend-net api

# Web Server (frontend only)
docker run -d \
  --name web \
  --network frontend-net \
  -p 8080:80 \
  nginx:latest

# Step 3: Test
# External: curl http://localhost:8080
# web -> api: both on frontend-net
# api -> postgres: both on backend-net
```

---

## 🛡️ Network Security

### Firewall Rules
```bash
# Restrict access between containers
docker run -d \
  --name secure-app \
  --network mynet \
  --ipc private \
  myapp:latest
```

### Encrypted Communication
```bash
# Use TLS for container communication
docker run -d \
  --name app \
  -v /certs:/certs \
  myapp:latest
```

### Network Segmentation
```bash
# Create isolated networks for different tiers
docker network create dmz
docker network create internal
docker network create database

# Each tier on separate network
# Define what can access what
```

---

## ✅ Checklist

- [ ] Understand Docker network types
- [ ] Create and manage bridge networks
- [ ] Connect containers to networks
- [ ] Use automatic DNS resolution
- [ ] Configure port mapping
- [ ] Understand overlay networks
- [ ] Set up multi-tier applications
- [ ] Know network security practices
- [ ] Can troubleshoot network issues
- [ ] Familiar with service discovery

