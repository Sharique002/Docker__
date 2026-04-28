# Practical 3: Creating Dockerfiles - Complete Guide

## 📋 Overview
Master the art of creating Dockerfiles to containerize applications with best practices.

---

## 🏗️ Part 1: Dockerfile Basics

### What is a Dockerfile?
A Dockerfile is a text file containing instructions to build a Docker image. Each instruction creates a layer in the image.

### Basic Structure
```dockerfile
# Comment
FROM base_image:tag          # Start from base image
MAINTAINER author_name       # (Deprecated, use LABEL)
LABEL maintainer="email"     # Metadata
RUN command                  # Execute command in container
COPY host_path container_path  # Copy files from host to container
ADD source destination       # Like COPY but supports URLs and auto-extraction
WORKDIR /path                # Set working directory
ENV VAR=value                # Set environment variables
EXPOSE port                  # Document which ports the app listens to
VOLUME ["/path"]             # Create mount point
USER username                # Run commands as specific user
CMD ["executable"]           # Default command when container starts
ENTRYPOINT ["executable"]    # Configure container as executable
```

---

## 📚 Part 2: Dockerfile Instructions Explained

### 1. FROM - Base Image
```dockerfile
# Official images from Docker Hub
FROM python:3.9-slim
FROM node:16-alpine
FROM ubuntu:20.04
FROM nginx:latest

# From specific registry
FROM gcr.io/project/image:tag

# Multi-stage: use previous build stage as base
FROM builder as base
```

### 2. RUN - Execute Commands
```dockerfile
# Single command
RUN apt-get update

# Multiple commands (better for layer optimization)
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean

# Execute shell script
RUN /bin/bash -c 'echo "Hello"'
```

### 3. COPY - Copy Files
```dockerfile
# Copy single file
COPY script.sh /app/

# Copy directory
COPY src/ /app/src/

# Copy with ownership
COPY --chown=appuser:appuser app/ /app/
```

### 4. ADD - Add Files (with special features)
```dockerfile
# Copy from host
ADD local_file /app/

# Extract tar automatically
ADD archive.tar.gz /app/

# Download from URL
ADD https://example.com/file.tar.gz /app/
```

### 5. WORKDIR - Set Working Directory
```dockerfile
WORKDIR /app
RUN npm install     # Runs in /app
COPY . .            # Copies to /app
```

### 6. ENV - Set Environment Variables
```dockerfile
ENV APP_ENV=production
ENV DB_HOST=localhost \
    DB_PORT=5432 \
    DB_NAME=mydb
```

### 7. EXPOSE - Document Ports
```dockerfile
EXPOSE 8080
EXPOSE 3000
EXPOSE 8080 9090 5000  # Multiple ports
```

### 8. VOLUME - Create Mount Points
```dockerfile
VOLUME ["/data"]
VOLUME ["/logs", "/cache"]
```

### 9. USER - Set User Context
```dockerfile
# Create user
RUN useradd -m appuser
USER appuser

# Or use pre-existing user
USER root
```

### 10. CMD - Default Command
```dockerfile
# Exec form (recommended)
CMD ["python", "app.py"]

# Shell form
CMD python app.py

# Default arguments for ENTRYPOINT
CMD ["--default", "args"]
```

### 11. ENTRYPOINT - Container Executable
```dockerfile
# Exec form (recommended)
ENTRYPOINT ["python", "app.py"]

# Shell form
ENTRYPOINT python app.py

# Combined with CMD
ENTRYPOINT ["python"]
CMD ["app.py"]
```

---

## 🔨 Part 3: Practical Dockerfile Examples

### Example 1: Python Flask Application
```dockerfile
# Multi-stage build
FROM python:3.9-slim as builder

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Final stage
FROM python:3.9-slim

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application
COPY . .

# Set environment
ENV PATH=/root/.local/bin:$PATH \
    PYTHONUNBUFFERED=1

# Create non-root user
RUN useradd -m -u 1000 appuser
USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

### Example 2: Node.js Application
```dockerfile
FROM node:16-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application
COPY . .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js

EXPOSE 3000

CMD ["node", "index.js"]
```

### Example 3: Go Application
```dockerfile
# Build stage
FROM golang:1.18-alpine as builder

WORKDIR /app

COPY . .

RUN go mod download && \
    CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Runtime stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=builder /app/app .

EXPOSE 8080

CMD ["./app"]
```

### Example 4: Multi-Service Application
```dockerfile
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

# Install multiple services
RUN apt-get update && apt-get install -y \
    nginx \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY . .

RUN pip3 install -r requirements.txt

EXPOSE 80 5000

# Start both services
CMD service nginx start && python3 app.py
```

---

## 🎯 Part 4: Best Practices

### 1. Use Specific Base Image Versions
```dockerfile
# ❌ Bad - gets latest, which might break
FROM python

# ✅ Good - specific version
FROM python:3.9-slim-bullseye
```

### 2. Minimize Layer Size
```dockerfile
# ❌ Bad - creates multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# ✅ Good - single layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean
```

### 3. Use .dockerignore
Create `.dockerignore` file:
```
.git
.gitignore
.DS_Store
node_modules
*.log
.env
```

### 4. Multi-Stage Build (Reduce Image Size)
```dockerfile
# ✅ Good - reduces final image size
FROM golang:1.18 as builder
RUN go build -o app .

FROM alpine:latest
COPY --from=builder /app /app
```

### 5. Use Alpine Images
```dockerfile
# ❌ Large - 169MB
FROM ubuntu:20.04

# ✅ Small - 5MB
FROM alpine:latest
```

### 6. Non-Root User
```dockerfile
RUN useradd -m -u 1000 appuser
USER appuser
```

### 7. Use LABEL for Metadata
```dockerfile
LABEL version="1.0" \
      description="Python Flask App" \
      maintainer="your@email.com"
```

### 8. Add Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1
```

### 9. Cache Optimization
```dockerfile
# Put frequently changing files last
FROM python:3.9
COPY requirements.txt .
RUN pip install -r requirements.txt  # Cached
COPY . .                               # Changes often, cached separately
```

### 10. Security Practices
```dockerfile
# Keep base image updated
FROM python:3.9-slim

# Install security updates
RUN apt-get update && apt-get upgrade -y && apt-get clean

# Use specific package versions
RUN pip install Flask==2.0.0

# Don't run as root
USER appuser
```

---

## 🚀 Part 5: Building and Testing

### Build Image
```bash
# Build with tag
docker build -t myapp:1.0 .

# Build with custom Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t myapp:1.0 .

# Build with progress output
docker build --progress=plain -t myapp:1.0 .
```

### Build Arguments (ARG)
```dockerfile
ARG BASE_IMAGE=python:3.9-slim
ARG APP_VERSION=1.0

FROM ${BASE_IMAGE}

LABEL version=${APP_VERSION}
```

```bash
# Build with custom arguments
docker build \
    --build-arg BASE_IMAGE=python:3.9 \
    --build-arg APP_VERSION=2.0 \
    -t myapp:2.0 .
```

### Test Image
```bash
# Run container
docker run -it myapp:1.0 bash

# Check image size
docker images myapp:1.0

# Inspect layers
docker history myapp:1.0

# Check for vulnerabilities
docker scan myapp:1.0
```

---

## 🎓 Hands-On Exercises

### Exercise 1: Simple Web Server
Create a Dockerfile for a simple Python web server:

```dockerfile
FROM python:3.9-slim

WORKDIR /app

RUN pip install flask

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

```bash
# Build and run
docker build -t web-server:1.0 .
docker run -d -p 5000:5000 web-server:1.0
```

### Exercise 2: Multi-Stage Build
Create a optimized Go application Dockerfile

```dockerfile
FROM golang:1.18 as builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM alpine:latest
COPY --from=builder /app/app /app/app
EXPOSE 8080
CMD ["./app/app"]
```

### Exercise 3: Node.js with Dependencies
Create a Node application with proper dependency caching

```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

---

## 📊 Dockerfile Commands Reference

| Instruction | Purpose | Notes |
|-----------|---------|-------|
| FROM | Base image | Required, should be first |
| RUN | Execute command | Creates new layer |
| COPY | Copy files | Recommended over ADD |
| ADD | Add files | Supports URLs and extraction |
| WORKDIR | Set directory | Changes all subsequent paths |
| ENV | Environment variable | Accessible in containers |
| EXPOSE | Document ports | Doesn't actually publish |
| VOLUME | Mount point | Create persistent storage |
| USER | Set user context | Security best practice |
| CMD | Default command | Overridable |
| ENTRYPOINT | Container executable | Fixed entry point |
| LABEL | Metadata | Version, author, etc |
| HEALTHCHECK | Health check | Monitor container health |

---

## ✅ Checklist

- [ ] Understand all Dockerfile instructions
- [ ] Know difference between CMD and ENTRYPOINT
- [ ] Can optimize layers and image size
- [ ] Familiar with multi-stage builds
- [ ] Use Alpine images for size efficiency
- [ ] Apply security best practices
- [ ] Can write optimized Dockerfiles
- [ ] Completed all 3 exercises

