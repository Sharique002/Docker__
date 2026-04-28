# Practical 2: Docker Hub - Account Creation, Pushing Images & Saving Changes

## 📋 Overview
Learn how to create a Docker Hub account, push containers, and save changes to container images.

---

## 🔑 Part 1: Creating a Docker Hub Account

### Step 1: Sign Up
1. Visit: https://hub.docker.com/
2. Click "Sign Up"
3. Choose your plan (Free tier is sufficient for learning)
4. Fill in:
   - Username (Docker ID) - this will be used in image names
   - Email
   - Password
5. Verify your email

### Step 2: Create Personal Access Token (Recommended)
1. Login to Docker Hub
2. Go to Account Settings → Security
3. Click "New Access Token"
4. Name it (e.g., "docker-cli")
5. Copy the token and save it securely

---

## 🐳 Part 2: Docker Login & Setup

### Login via CLI
```bash
# Login with Docker Hub credentials
docker login

# Enter username: your_docker_id
# Enter password: (will be hidden)
# Or with token:
docker login --username your_docker_id --password-stdin < token.txt

# Logout
docker logout
```

### Verify Login
```bash
cat ~/.docker/config.json  # Linux/Mac
# or
type %USERPROFILE%\.docker\config.json  # Windows
```

---

## 📤 Part 3: Pushing Images to Docker Hub

### Step 1: Build an Image

#### Option A: Using existing app
```bash
# Navigate to your project
cd /path/to/your/docker/project

# Build image with Docker Hub format
docker build -t your_docker_id/my-app:1.0 .

# List images
docker images
```

#### Option B: Create a simple app to push

**Create app directory:**
```bash
mkdir my-app && cd my-app
```

**Create Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY . .

RUN pip install flask

CMD ["python", "app.py"]
```

**Create app.py:**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Docker Hub!'

@app.route('/api/status')
def status():
    return {'status': 'running'}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Create requirements.txt:**
```
Flask==2.3.0
```

### Step 2: Build and Push
```bash
# Build the image
docker build -t your_docker_id/my-app:1.0 .

# Verify image created
docker images | grep my-app

# Push to Docker Hub
docker push your_docker_id/my-app:1.0

# Check Docker Hub - your image should be visible in your repository
```

### Step 3: Create Multiple Tags
```bash
# Tag image as latest
docker tag your_docker_id/my-app:1.0 your_docker_id/my-app:latest

# Push latest tag
docker push your_docker_id/my-app:latest

# View all tags on Docker Hub website
```

---

## 💾 Part 4: Saving Changes to Container

### Method 1: Docker Commit (Create Image from Running Container)

#### Step 1: Run a Container
```bash
# Run container from ubuntu
docker run -it --name my-dev ubuntu:20.04 /bin/bash

# Inside container, make changes:
# apt-get update
# apt-get install -y curl wget git
# echo "Added packages" > /etc/myconfig.txt
# exit
```

#### Step 2: Commit Changes
```bash
# Create image from container
docker commit my-dev your_docker_id/ubuntu-custom:1.0

# Verify image created
docker images

# Remove old container
docker rm my-dev

# Run container from new image
docker run -it --name test-custom your_docker_id/ubuntu-custom:1.0 /bin/bash
# curl --version  # Should work now
# exit
```

#### Step 3: Push Custom Image
```bash
docker push your_docker_id/ubuntu-custom:1.0
```

### Method 2: Dockerfile (Recommended)

**Better approach - use Dockerfile:**

```dockerfile
FROM ubuntu:20.04

RUN apt-get update && \
    apt-get install -y curl wget git && \
    apt-get clean

RUN echo "Custom Ubuntu Image" > /etc/myconfig.txt

WORKDIR /workspace

CMD ["/bin/bash"]
```

**Build and push:**
```bash
docker build -t your_docker_id/ubuntu-custom:2.0 .
docker push your_docker_id/ubuntu-custom:2.0
```

---

## 🔄 Workflow: Complete Push Example

### Full Example Workflow
```bash
# 1. Create directory
mkdir docker-hello-app && cd docker-hello-app

# 2. Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:16-alpine

WORKDIR /app

COPY . .

RUN npm install

EXPOSE 3000

CMD ["node", "server.js"]
EOF

# 3. Create server.js
cat > server.js << 'EOF'
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello from Docker!\n');
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
EOF

# 4. Create package.json
cat > package.json << 'EOF'
{
  "name": "docker-hello",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  }
}
EOF

# 5. Build image
docker build -t your_docker_id/docker-hello:1.0 .

# 6. Test locally
docker run -d -p 3000:3000 --name test your_docker_id/docker-hello:1.0
# Visit: http://localhost:3000
docker stop test
docker rm test

# 7. Push to Docker Hub
docker push your_docker_id/docker-hello:1.0

# 8. Verify on Docker Hub website
```

---

## 🎯 Hands-On Exercises

### Exercise 1: Create and Push Your First Image
```bash
# 1. Create app directory
mkdir my-first-app && cd my-first-app

# 2. Create Dockerfile
# 3. Create app files
# 4. Build: docker build -t your_id/my-app:1.0 .
# 5. Test: docker run -d -p 8080:8080 your_id/my-app:1.0
# 6. Push: docker push your_id/my-app:1.0
```

### Exercise 2: Create Custom Base Image
```bash
# 1. Run base image with modifications
docker run -it --name mybase ubuntu:20.04 /bin/bash
# apt-get update && apt-get install -y vim curl

# 2. Commit changes
docker commit mybase your_id/mybase:1.0

# 3. Push
docker push your_id/mybase:1.0

# 4. Use in new Dockerfile
# FROM your_id/mybase:1.0
# ...
```

### Exercise 3: Version Management
```bash
# Create multiple versions
docker build -t your_id/myapp:1.0 .
docker build -t your_id/myapp:1.1 .
docker build -t your_id/myapp:latest .

# Push all versions
docker push your_id/myapp:1.0
docker push your_id/myapp:1.1
docker push your_id/myapp:latest

# Verify on Docker Hub
```

---

## 📊 Useful Docker Hub Commands

```bash
# Search for images on Docker Hub
docker search nginx

# Pull specific image version
docker pull ubuntu:20.04

# Pull all versions of an image
docker pull nginx  # Gets latest
docker pull nginx:alpine  # Specific variant

# View image details
docker inspect your_id/myapp:1.0

# Remove local image
docker rmi your_id/myapp:1.0

# Clean up (remove dangling images)
docker image prune
```

---

## 🔐 Best Practices

### Security
```bash
# Use personal access tokens instead of passwords
# Never commit Docker credentials to git

# Don't store secrets in images
# Use environment variables or secret management

# Scan images for vulnerabilities
docker scan your_id/myapp:1.0

# Use minimal base images
# FROM python:3.9-slim  # Instead of python:3.9
```

### Versioning
```bash
# Use semantic versioning
docker build -t your_id/app:1.0.0 .
docker build -t your_id/app:1.0 .
docker build -t your_id/app:latest .

# Always tag with specific version, not just latest
```

### Tagging Conventions
```bash
# Good tagging:
your_id/appname:1.0.0
your_id/appname:v1.0.0
your_id/appname:production
your_id/appname:staging

# Link versions
docker tag your_id/appname:1.0.0 your_id/appname:latest
```

---

## ✅ Checklist

- [ ] Docker Hub account created
- [ ] Docker CLI logged in
- [ ] Built first image locally
- [ ] Pushed image to Docker Hub
- [ ] Image visible in Docker Hub repository
- [ ] Pulled image on another machine successfully
- [ ] Created custom image from running container
- [ ] Used `docker commit` to save changes
- [ ] Practiced versioning and tagging

