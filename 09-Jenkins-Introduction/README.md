# Practical 9: Introduction to Continuous Integration with Jenkins

## 📋 Overview
Learn CI/CD concepts and Jenkins for automated building, testing, and deployment.

---

## 🔄 Part 1: Continuous Integration Concepts

### What is CI/CD?
**CI (Continuous Integration)**: Automatically build and test code changes
**CD (Continuous Deployment)**: Automatically deploy to production

### CI/CD Pipeline Flow
```
Code Push → Trigger → Build → Test → Deploy → Monitor
   ↓         ↓        ↓      ↓       ↓        ↓
Git Repo  Jenkins  Docker  Tests  Registry  Production
```

### Benefits
1. **Faster Feedback**: Know about issues immediately
2. **Automation**: Reduce manual errors
3. **Consistency**: Same process every time
4. **Quality**: Catch issues early
5. **Speed**: Deploy changes faster

### Common CI/CD Tools
- Jenkins: Open-source, self-hosted
- GitLab CI/CD: Integrated with GitLab
- GitHub Actions: Native to GitHub
- CircleCI: Cloud-based
- Travis CI: Cloud-based

---

## 🧩 Part 2: Jenkins Architecture

### Jenkins Components
```
┌─────────────────────────────┐
│    Jenkins Master/Server     │
├─────────────────────────────┤
│ • Orchestrates jobs         │
│ • Schedules builds          │
│ • Manages plugins           │
│ • Stores configurations     │
└─────────────────────────────┘
        ↓          ↓         ↓
    ┌────────┐ ┌────────┐ ┌────────┐
    │ Agent 1│ │ Agent 2│ │ Agent 3│
    │ Node   │ │ Node   │ │ Node   │
    └────────┘ └────────┘ └────────┘
```

### Key Concepts

#### Job
A unit of work that Jenkins executes (build, test, deploy)

#### Pipeline
Series of steps/stages to execute

#### Trigger
Event that starts a job (webhook, schedule, manual)

#### Build
Compilation of source code

#### Test
Automated tests to verify functionality

#### Artifact
Output of a build (JAR, Docker image, etc.)

#### Agent/Slave
Machine that runs jobs under Jenkins control

---

## 📦 Part 3: Jenkins Installation

### Using Docker
```bash
# Pull Jenkins image
docker pull jenkins/jenkins:lts

# Create volume for persistence
docker volume create jenkins-home

# Run Jenkins
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial password
docker logs jenkins | grep "initialAdminPassword"

# Access Jenkins
# http://localhost:8080
```

### Using Docker Compose
```yaml
version: '3.9'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins-home:/var/jenkins_home
    environment:
      - JENKINS_OPTS=-Djavax.net.debug=ssl:handshake
    restart: unless-stopped

volumes:
  jenkins-home:
```

### Initial Setup
1. Open http://localhost:8080
2. Enter initial admin password from logs
3. Install suggested plugins
4. Create admin user
5. Configure Jenkins URL

---

## 🔨 Part 4: Creating Jenkins Jobs

### Simple Freestyle Job

#### Step 1: Create New Job
1. Dashboard → New Item
2. Name: "Hello World"
3. Select: Freestyle project
4. Click OK

#### Step 2: Configure Job
1. **General Tab**
   - Description: "My first Jenkins job"

2. **Build Tab**
   - Add build step: Execute shell
   - Command: `echo "Hello from Jenkins"`

3. **Post-build Tab**
   - Add post-build action: Archive artifacts
   - Files to archive: `*.*`

#### Step 3: Run Job
- Click "Build Now"
- View console output

### Advanced: Docker Build Job

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/repo.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker run --rm myapp:${BUILD_NUMBER} npm test'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker push user/myapp:${BUILD_NUMBER}'
            }
        }
    }
}
```

---

## 🔧 Part 5: Jenkins Plugins

### Essential Plugins
```bash
# Install via Jenkins UI:
# Manage Jenkins → Plugin Manager → Available

# Popular plugins:
- Docker plugin: Build and run Docker containers
- GitHub plugin: GitHub integration
- Pipeline plugin: Pipeline support
- Credentials plugin: Manage secrets
- Email Extension: Email notifications
- Blue Ocean: Modern UI
```

### Installing Plugins
1. Manage Jenkins → Plugin Manager
2. Available tab
3. Search for plugin name
4. Check box and click "Install without restart"
5. Or "Download now and install after restart"

---

## 🎯 Part 6: Hands-On Examples

### Example 1: Simple Build Job

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'echo "Build step"'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'echo "Test step"'
            }
        }
    }
    
    post {
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

### Example 2: Docker Build and Push

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = credentials('docker-credentials')
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t myapp:latest .'
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker run --rm myapp:latest npm test'
            }
        }
        
        stage('Push') {
            steps {
                sh '''
                    echo $DOCKER_REGISTRY_PSW | docker login -u $DOCKER_REGISTRY_USR --password-stdin
                    docker tag myapp:latest username/myapp:latest
                    docker push username/myapp:latest
                '''
            }
        }
    }
}
```

### Example 3: GitHub Integration

```groovy
pipeline {
    agent any
    
    triggers {
        githubPush()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
    }
}
```

---

## 📋 Part 7: Jenkins Configuration

### Configure System
1. Manage Jenkins → Configure System
2. Common settings:
   - Jenkins Location (URL)
   - System Admin email address
   - Email notification settings

### Configure Global Security
1. Manage Jenkins → Configure Global Security
2. Options:
   - Authorization strategy
   - CSRF protection
   - User signup

### Configure Nodes/Agents
1. Manage Jenkins → Manage Nodes and Clouds
2. Create new node
3. Configure SSH connection to agent
4. Add labels for job assignment

---

## 🧪 Hands-On Exercise: Complete Setup

### Step 1: Start Jenkins
```bash
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-home:/var/jenkins_home \
  jenkins/jenkins:lts
```

### Step 2: Initial Configuration
- Access Jenkins at http://localhost:8080
- Get admin password from logs
- Setup admin user

### Step 3: Create GitHub Integration
1. GitHub → Settings → Developer settings → Personal access tokens
2. Create token with repo access
3. Jenkins → Credentials → Add credentials
4. Paste GitHub token

### Step 4: Create Pipeline Job
1. New Item → Pipeline
2. Pipeline definition: Pipeline script from SCM
3. SCM: Git
4. Repository URL: Your GitHub repo
5. Build trigger: GitHub hook trigger for GITScm polling

### Step 5: Push Changes
```bash
git push origin main
# Automatically triggers Jenkins build
```

---

## 🔐 Secrets Management

### Store Credentials
```groovy
pipeline {
    agent any
    
    environment {
        DB_PASSWORD = credentials('db-password')
        API_KEY = credentials('api-key')
    }
    
    stages {
        stage('Deploy') {
            steps {
                sh 'echo "Using credentials"'
                // Variables available as $DB_PASSWORD, $API_KEY
            }
        }
    }
}
```

### Add Credentials
1. Jenkins → Manage Credentials
2. Global domain → Add Credentials
3. Kind: Username with password (or Secret text)
4. Fill in details
5. Use in pipeline: `credentials('credential-id')`

---

## ✅ Checklist

- [ ] Understand CI/CD concepts
- [ ] Know Jenkins architecture
- [ ] Install Jenkins using Docker
- [ ] Access Jenkins web interface
- [ ] Create simple freestyle job
- [ ] Understand pipeline syntax
- [ ] Create Docker build job
- [ ] Integrate with GitHub
- [ ] Set up webhooks
- [ ] Store credentials securely
- [ ] View build history and logs

