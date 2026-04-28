# Practical 7: Container Orchestration & Docker Swarm

## 📋 Overview
Learn container orchestration concepts and master Docker Swarm for managing container clusters.

---

## 🏛️ Part 1: Container Orchestration Basics

### What is Container Orchestration?
Orchestration automates deployment, management, and scaling of containers across multiple machines.

### Key Problems Solved
1. **Deployment**: Automatically place containers on appropriate nodes
2. **Scaling**: Add/remove container instances based on demand
3. **Failover**: Restart failed containers automatically
4. **Updates**: Rolling deployments without downtime
5. **Networking**: Service discovery and load balancing
6. **Storage**: Persistent storage management

### Orchestration Platforms
| Tool | Best For | Complexity |
|------|----------|-----------|
| Docker Swarm | Simple deployments, learning | Easy |
| Kubernetes | Enterprise, complex workloads | Hard |
| Docker Compose | Single-machine multi-container | Very easy |
| ECS (AWS) | AWS ecosystem | Medium |

---

## 🐳 Part 2: Docker Swarm Fundamentals

### What is Docker Swarm?
Docker Swarm is Docker's native container orchestration platform. It turns a group of Docker engines into one virtual Docker engine.

### Swarm Architecture
```
┌─────────────────────────────────────────┐
│        Docker Swarm Cluster              │
├─────────────────────────────────────────┤
│                                          │
│  ┌──────────────┐  ┌──────────────┐    │
│  │Manager Node 1│  │Manager Node 2│    │
│  │(Leader)      │  │              │    │
│  └──────────────┘  └──────────────┘    │
│         ↓                  ↓             │
│  ┌──────────────┐  ┌──────────────┐    │
│  │ Worker Node 1│  │ Worker Node 2│    │
│  └──────────────┘  └──────────────┘    │
│                                          │
└─────────────────────────────────────────┘
```

### Components
1. **Manager Node**: Orchestrates and manages the swarm
2. **Worker Node**: Runs tasks/containers
3. **Service**: Declarative definition of tasks to run
4. **Task**: Running container instance
5. **Load Balancer**: Routes traffic to tasks

---

## 🚀 Part 3: Setting Up Docker Swarm

### Initialize Swarm
```bash
# Initialize swarm on first node (becomes manager)
docker swarm init

# Output shows token to join
# Copy this for joining nodes
```

### Join Additional Nodes
```bash
# On other machines, run the token from init output:
docker swarm join --token SWMTKN-xxx-xxx 192.168.1.100:2377

# Verify nodes joined
docker node ls
```

### Get Join Tokens
```bash
# Get worker token
docker swarm join-token worker

# Get manager token
docker swarm join-token manager

# Rotate tokens
docker swarm join-token --rotate worker
```

### Remove Nodes
```bash
# Leave swarm (from worker node)
docker swarm leave

# Remove node (from manager)
docker node rm node_id

# Force remove
docker node rm --force node_id
```

---

## 📦 Part 4: Docker Swarm Services

### Create Service
```bash
# Simple service
docker service create --name web nginx:latest

# With replicas
docker service create --name web --replicas 3 nginx:latest

# With port mapping
docker service create \
  --name web \
  --replicas 3 \
  -p 8080:80 \
  nginx:latest

# With resource limits
docker service create \
  --name app \
  --replicas 2 \
  --limit-cpu 0.5 \
  --limit-memory 512M \
  myapp:latest

# With environment variables
docker service create \
  --name db \
  -e MYSQL_ROOT_PASSWORD=root123 \
  mysql:latest
```

### Manage Services
```bash
# List services
docker service ls

# Get service details
docker service inspect web

# View service logs
docker service logs web

# Scale service
docker service scale web=5

# Update service
docker service update --replicas 3 web

# Update image
docker service update --image nginx:1.21 web

# Remove service
docker service rm web
```

### Rolling Updates
```bash
# Update with rolling update strategy
docker service update \
  --image myapp:2.0 \
  --update-delay 10s \
  --update-parallelism 2 \
  app

# Rollback to previous version
docker service rollback app
```

---

## 🌐 Part 5: Networking in Swarm

### Ingress Network (Overlay)
```bash
# Automatically created, handles traffic routing
# Port published on any node routes to any replica

docker service create \
  --name web \
  --replicas 3 \
  -p 8080:80 \
  nginx:latest

# Access from any node: http://nodeIP:8080
```

### Custom Overlay Network
```bash
# Create overlay network
docker network create \
  --driver overlay \
  --scope swarm \
  backend

# Create service on network
docker service create \
  --name app \
  --network backend \
  myapp:latest

docker service create \
  --name db \
  --network backend \
  postgres:latest

# Services communicate via hostname
# app can connect to: postgres://db:5432
```

---

## 🎯 Part 6: Hands-On Swarm Examples

### Example 1: Create Multi-Node Cluster
```bash
# Node 1: Initialize
docker swarm init

# Get token
TOKEN=$(docker swarm join-token -q worker)

# Node 2, 3, etc: Join
docker swarm join --token $TOKEN manager_ip:2377

# Verify
docker node ls
```

### Example 2: Deploy Multi-Service Stack
```bash
# Create network
docker network create -d overlay services

# Create database
docker service create \
  --name postgres \
  --network services \
  -e POSTGRES_PASSWORD=root \
  postgres:latest

# Create web app
docker service create \
  --name webapp \
  --network services \
  -p 8080:80 \
  --replicas 3 \
  mywebapp:latest

# List services
docker service ls
```

### Example 3: Scale and Update
```bash
# Monitor service
watch docker service ls

# Scale up
docker service scale webapp=5

# Update to new version
docker service update --image mywebapp:2.0 webapp

# Scale down
docker service scale webapp=2
```

---

## 🔧 Part 7: Global Services and Constraints

### Global Service (run on every node)
```bash
# Create global service (monitoring agent on each node)
docker service create \
  --mode global \
  --name monitoring \
  prometheus:latest
```

### Replicated Service (specific count)
```bash
# Default mode - run specified number of replicas
docker service create \
  --name web \
  --replicas 3 \
  nginx:latest
```

### Placement Constraints
```bash
# Constrain to specific node
docker service create \
  --name db \
  --constraint node.hostname==production \
  postgres:latest

# Constrain by label
docker node update --label-add env=production node-1

docker service create \
  --name critical-app \
  --constraint node.labels.env==production \
  myapp:latest
```

---

## 📊 Docker Stack (Services with Compose)

### Convert Compose to Stack
```yaml
# docker-compose.yml
version: '3.9'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    replicas: 3

  db:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD: root

networks:
  default:
    driver: overlay
```

### Deploy Stack
```bash
# Deploy from compose file
docker stack deploy -c docker-compose.yml myapp

# List stacks
docker stack ls

# List services in stack
docker stack services myapp

# View stack details
docker stack inspect myapp

# Remove stack
docker stack rm myapp
```

---

## 🧪 Hands-On Exercise: Complete Setup

### Step 1: Initialize Swarm
```bash
docker swarm init
docker node ls
```

### Step 2: Create Overlay Network
```bash
docker network create -d overlay mystack
```

### Step 3: Create Services
```bash
# Database
docker service create \
  --name postgres \
  --network mystack \
  --env POSTGRES_PASSWORD=root \
  postgres:latest

# Web App
docker service create \
  --name webapp \
  --network mystack \
  --replicas 3 \
  -p 8080:80 \
  myapp:latest
```

### Step 4: Monitor
```bash
docker service ls
docker service ps webapp
docker service logs webapp
```

### Step 5: Scale
```bash
docker service scale webapp=5
watch docker service ps webapp
```

---

## ✅ Checklist

- [ ] Understand container orchestration concepts
- [ ] Initialize Docker Swarm
- [ ] Join worker nodes to swarm
- [ ] Create and manage services
- [ ] Scale services up/down
- [ ] Deploy multi-service stacks
- [ ] Configure overlay networks
- [ ] Use service discovery
- [ ] Perform rolling updates
- [ ] Know when to use Swarm vs Kubernetes

