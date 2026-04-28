# Practical 1: Installing Docker on Windows/Linux, Hyper-V Concept & Basic Docker Commands

## 📋 Overview
This practical covers Docker installation on different platforms, understanding Hyper-V, and learning essential Docker commands.

---

## 🔧 Part 1: Installing Docker

### Windows Installation

#### Prerequisites:
- Windows 10/11 Pro, Enterprise, or Education edition
- 4GB RAM minimum (8GB recommended)
- BIOS virtualization enabled
- Hyper-V enabled

#### Steps:
1. **Download Docker Desktop for Windows**
   - Visit: https://www.docker.com/products/docker-desktop
   - Download the latest version

2. **Install Docker Desktop**
   ```bash
   # Run the installer
   Docker Desktop Installer.exe
   ```

3. **Enable WSL 2 (Windows Subsystem for Linux 2)**
   ```bash
   # Open PowerShell as Administrator and run:
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
   
   # Restart your computer
   # Then set WSL 2 as default:
   wsl --set-default-version 2
   ```

4. **Verify Installation**
   ```bash
   docker --version
   docker run hello-world
   ```

### Linux Installation (Ubuntu/Debian)

#### Steps:
```bash
# Update package manager
sudo apt-get update

# Install Docker
sudo apt-get install docker.io -y

# Add current user to docker group (optional, to avoid sudo)
sudo usermod -aG docker $USER
newgrp docker

# Verify Installation
docker --version
docker run hello-world
```

---

## 🖥️ Part 2: Understanding Hyper-V

### What is Hyper-V?
Hyper-V is a Microsoft virtualization platform that allows you to create and manage virtual machines on Windows.

### Key Concepts:
- **Virtual Machine**: Emulated computer running on your physical machine
- **Hypervisor**: Software that manages virtual machines
- **Type 1 Hypervisor**: Runs directly on hardware (Hyper-V)
- **Type 2 Hypervisor**: Runs on top of OS (VirtualBox)

### Docker and Hyper-V:
- Docker Desktop on Windows uses Hyper-V to run a lightweight Linux VM
- This Linux VM runs the Docker daemon
- Containers run inside this Linux VM

### Enable Hyper-V (Windows):
```bash
# PowerShell (as Administrator)
Enable-WindowsOptionalFeature -Online -FeatureName Hyper-V -All

# Or via Control Panel:
# Control Panel > Programs > Programs and Features > Turn Windows features on or off
# Check: Hyper-V
```

---

## ⚙️ Part 3: Basic Docker Commands

### 1. Image Commands

#### Pull an Image
```bash
# Pull an image from Docker Hub
docker pull ubuntu:20.04

# List all images
docker images

# Remove an image
docker rmi ubuntu:20.04
```

#### Build an Image
```bash
# Build from Dockerfile
docker build -t my-app:1.0 .

# Build with custom Dockerfile path
docker build -f /path/to/Dockerfile -t my-app:1.0 .
```

### 2. Container Commands

#### Run a Container
```bash
# Interactive container
docker run -it ubuntu:20.04 bash

# Detached container (background)
docker run -d --name my-container nginx

# With port mapping
docker run -d -p 8080:80 --name webserver nginx

# With volume mount
docker run -d -v /host/path:/container/path --name my-container ubuntu

# With environment variables
docker run -d -e DB_PASSWORD=secret --name app myapp:1.0
```

#### List Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List with more details
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.Status}}"
```

#### Container Management
```bash
# Start a stopped container
docker start container_id

# Stop a running container
docker stop container_id

# Pause a container
docker pause container_id

# Unpause a container
docker unpause container_id

# Remove a container
docker rm container_id

# Remove all stopped containers
docker container prune

# View logs
docker logs container_id

# Follow logs (real-time)
docker logs -f container_id

# Execute command in running container
docker exec -it container_id bash

# Inspect container details
docker inspect container_id
```

### 3. Other Useful Commands

#### System Information
```bash
# Docker version
docker version

# Docker info
docker info

# Docker disk usage
docker system df

# Clean up unused resources
docker system prune
```

#### Save and Load Images
```bash
# Save image to tar file
docker save myimage:1.0 > myimage.tar

# Load image from tar file
docker load < myimage.tar
```

---

## 🎯 Hands-On Exercises

### Exercise 1: Run Your First Container
```bash
# Pull and run Ubuntu
docker pull ubuntu:20.04
docker run -it ubuntu:20.04 /bin/bash

# Inside container:
# apt-get update
# apt-get install curl
# curl https://www.google.com
# exit
```

### Exercise 2: Run a Web Server
```bash
# Run Nginx web server
docker run -d -p 8080:80 --name my-web nginx

# Check if it's running
docker ps

# View logs
docker logs my-web

# Access the web server
# Open browser: http://localhost:8080

# Stop and remove
docker stop my-web
docker rm my-web
```

### Exercise 3: Multiple Containers
```bash
# Run multiple containers
docker run -d -p 8080:80 --name web1 nginx
docker run -d -p 8081:80 --name web2 nginx
docker run -d -p 8082:80 --name web3 nginx

# List all
docker ps

# Stop all
docker stop web1 web2 web3

# Remove all
docker rm web1 web2 web3
```

### Exercise 4: Inspect Container
```bash
# Run a container
docker run -d --name myapp -e NAME="Docker" nginx

# Inspect container
docker inspect myapp

# View specific info
docker inspect --format='{{.State.Running}}' myapp
docker inspect --format='{{.NetworkSettings.IPAddress}}' myapp
```

---

## 📊 Command Reference Table

| Command | Purpose |
|---------|---------|
| `docker pull [image]` | Download image from registry |
| `docker images` | List local images |
| `docker run [image]` | Create and start container |
| `docker ps` | List running containers |
| `docker stop [id]` | Stop container |
| `docker rm [id]` | Remove container |
| `docker logs [id]` | View container logs |
| `docker exec [id] [cmd]` | Run command in container |
| `docker build -t [name]` | Build image from Dockerfile |

---

## 🧪 Troubleshooting

### Docker daemon not running
```bash
# Windows: Restart Docker Desktop
# Linux: 
sudo systemctl restart docker
```

### Permission denied error (Linux)
```bash
# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

### Port already in use
```bash
# Change port mapping
docker run -d -p 9000:80 nginx

# Or stop container using port
docker stop container_id
```

---

## ✅ Checklist

- [ ] Docker installed and running
- [ ] Verified with `docker run hello-world`
- [ ] Understand Hyper-V basics
- [ ] Familiar with basic Docker commands
- [ ] Completed Exercise 1-4
- [ ] Can run and manage containers

