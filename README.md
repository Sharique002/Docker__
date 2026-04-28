# 🐳 Docker Learning Repository

A comprehensive guide to Docker with 10 detailed practicals covering everything from basic concepts to advanced CI/CD pipelines.

---

## 📚 Comprehensive Docker Practicals

### Beginner Level
1. **[Docker Installation & Basic Commands](01-Docker-Installation/README.md)** 
   - Installing Docker on Windows/Linux
   - Understanding Hyper-V
   - Basic Docker commands and operations

2. **[Docker Hub - Managing Images](02-Docker-Hub/README.md)**
   - Creating Docker Hub account
   - Pushing and pulling images
   - Saving container changes

3. **[Creating Dockerfiles](03-Dockerfile/README.md)**
   - Dockerfile instructions and syntax
   - Building optimized images
   - Multi-stage builds
   - Best practices

4. **[Docker Storage & Container Linking](04-Docker-Storage/README.md)**
   - Volume types (volumes, bind mounts, tmpfs)
   - Persistent data management
   - Linking containers together
   - Network communication

### Intermediate Level
5. **[Introduction to Microservices](05-Microservices/README.md)**
   - Microservices architecture concepts
   - Building multi-service applications
   - Service communication patterns
   - Scaling microservices

6. **[Docker Compose](06-Docker-Compose/README.md)**
   - Docker Compose installation
   - YAML file structure
   - Multi-container orchestration
   - WordPress deployment example

7. **[Container Orchestration & Docker Swarm](07-Docker-Swarm/README.md)**
   - Orchestration concepts
   - Setting up Docker Swarm
   - Managing services and replicas
   - Clustering and load balancing

8. **[Docker Networks](08-Docker-Networks/README.md)**
   - Network types (bridge, host, overlay, etc.)
   - Custom networks and service discovery
   - Multi-container communication
   - Network security

### Advanced Level
9. **[Jenkins & CI/CD Introduction](09-Jenkins-Introduction/README.md)**
   - CI/CD concepts and benefits
   - Jenkins architecture
   - Creating Jenkins jobs
   - Docker integration with Jenkins

10. **[Complete CI/CD Pipelines](10-CI-CD-Pipelines/README.md)**
    - Pipeline stages and architecture
    - Complete pipeline examples
    - Testing strategies
    - Deployment strategies (Blue-Green, Canary, Rolling)
    - Monitoring and logging

---

## 📖 Learning Path

**Start Here:** Begin with Practical 1 and follow the numbered sequence. Each practical builds on previous knowledge.

```
Installation → Hub → Dockerfile → Storage → Microservices
                                              ↓
    Compose ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ← ←
      ↓
    Swarm & Networks
      ↓
    Jenkins & CI/CD
```

---

## 🎯 Quick Navigation

| Practical | Level | Topics | Duration |
|-----------|-------|--------|----------|
| 01 | Beginner | Installation, Hyper-V, CLI | 2 hours |
| 02 | Beginner | Registry, Images, Tagging | 1.5 hours |
| 03 | Beginner | Dockerfiles, Optimization | 2 hours |
| 04 | Beginner | Storage, Networks, Linking | 1.5 hours |
| 05 | Intermediate | Microservices, Architecture | 2 hours |
| 06 | Intermediate | Compose, YAML, Multi-container | 2.5 hours |
| 07 | Intermediate | Swarm, Orchestration, Clustering | 2 hours |
| 08 | Intermediate | Networks, Discovery, Isolation | 1.5 hours |
| 09 | Advanced | Jenkins, CI/CD, Automation | 2 hours |
| 10 | Advanced | Pipelines, Testing, Deployment | 3 hours |

---

## 🚀 Getting Started

### Clone the Repository
```bash
git clone https://github.com/Sharique002/Docker__
cd Docker__
```

### Choose Your Starting Point
```bash
# For beginners
cd 01-Docker-Installation
cat README.md

# Or jump to a specific topic
cd 06-Docker-Compose
cat README.md
```

### Follow Each Practical
Each folder contains:
- **README.md**: Comprehensive theory and explanations
- **Examples**: Code snippets and configurations
- **Exercises**: Hands-on practical tasks

---

## 💡 Key Topics Covered

- ✅ Docker installation and setup
- ✅ Container and image management
- ✅ Dockerfile best practices
- ✅ Storage and data persistence
- ✅ Networking and service discovery
- ✅ Multi-container orchestration
- ✅ Docker Compose
- ✅ Docker Swarm clustering
- ✅ Microservices architecture
- ✅ CI/CD pipelines
- ✅ Jenkins automation
- ✅ Deployment strategies

---

## 📋 Prerequisites

- Docker installed (see Practical 01)
- Basic Linux/Unix command line knowledge
- Text editor or IDE
- Git for version control

---

## 🔗 Useful Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

## 📝 Notes

All practicals include:
- Theory and concepts
- Step-by-step instructions
- Multiple code examples
- Hands-on exercises
- Best practices
- Troubleshooting tips
- Checklists for verification

---

## 🤝 Contributing

Found improvements or errors? Feel free to contribute!

---

---

# Dockerized Python Flask Application

A simple, production-ready Python Flask web application containerized with Docker.

## 📋 Project Overview

This is a lightweight Flask web application that demonstrates how to containerize a Python application using Docker. The application runs a basic web server that responds to HTTP requests on port 5000.

### Features
- Simple Flask web server
- Health check endpoint
- Fully containerized with Docker
- Production-ready configuration

## 📁 Folder Structure

```
dockerfile/
│
├── app.py                 # Main Flask application
├── requirements.txt       # Python dependencies
├── Dockerfile            # Docker configuration
├── README.md            # Project documentation
└── .gitignore           # Git ignore rules
```

## 🐳 Docker Commands

### Build Docker Image

```bash
docker build -t flask-app .
```

This command builds a Docker image with the tag `flask-app` using the Dockerfile in the current directory.

### Run Docker Container

```bash
docker run -d -p 5000:5000 --name flask-container flask-app
```

Parameters explained:
- `-d`: Run container in detached mode (background)
- `-p 5000:5000`: Map port 5000 on host to port 5000 in container
- `--name flask-container`: Assign a name to the container
- `flask-app`: The image to use

## 🌐 Accessing the Application

Once the container is running, access the application at:

**http://localhost:5000**

### Available Endpoints

- `/` - Home page with welcome message
- `/health` - Health check endpoint (returns JSON status)

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd dockerfile
   ```

2. **Build the Docker image**
   ```bash
   docker build -t flask-app .
   ```

3. **Run the container**
   ```bash
   docker run -d -p 5000:5000 --name flask-container flask-app
   ```

4. **Verify it's running**
   ```bash
   docker ps
   ```

5. **View logs**
   ```bash
   docker logs flask-container
   ```

6. **Stop the container**
   ```bash
   docker stop flask-container
   ```

7. **Remove the container**
   ```bash
   docker rm flask-container
   ```

## 🛠️ Development

### Running Locally (without Docker)

1. Create a virtual environment:
   ```bash
   python -m venv venv
   ```

2. Activate the virtual environment:
   - Windows: `venv\Scripts\activate`
   - Linux/Mac: `source venv/bin/activate`

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Run the application:
   ```bash
   python app.py
   ```

## 📦 Dependencies

- Flask 3.0.0
- Werkzeug 3.0.1

## 📄 License

This project is open source and available for educational purposes.

## 👤 Author

Created as a demonstration of Docker containerization with Python Flask.

---

**Note**: This is a development configuration. For production deployments, consider using a production WSGI server like Gunicorn or uWSGI instead of Flask's built-in development server.
