# Practical 10: Understanding CI/CD Pipelines - Complete Guide

## 📋 Overview
Master building complete CI/CD pipelines for automated testing, building, and deployment.

---

## 🔄 Part 1: CI/CD Pipeline Concepts

### Pipeline Stages
```
┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐
│Source  │ → │ Build  │ → │ Test   │ → │ Deploy │ → │Monitor │
│Code    │   │ Image  │   │ Suite  │   │ Prod   │   │ Health │
└────────┘   └────────┘   └────────┘   └────────┘   └────────┘
    ↑                                                      ↓
    └──────────────── Feedback Loop ────────────────────→
```

### Key Components
1. **Source Control**: Git repository (GitHub, GitLab, Bitbucket)
2. **Build**: Compile code, create Docker image
3. **Test**: Unit tests, integration tests, security scanning
4. **Artifact Repository**: Store built images
5. **Deployment**: Dev → Staging → Production
6. **Monitoring**: Track application health and metrics

---

## 🏗️ Part 2: Pipeline Architecture

### Single Branch Pipeline
```
Git Push → Build → Test → Deploy to Dev → Manual Approval → Deploy to Prod
```

### Multi-Branch Pipeline
```
          ├→ Build → Test → Deploy to Dev
Git Push ─┤
          └→ Build → Test → Deploy to Staging → Approval → Deploy to Prod
```

### Best Practices
1. **Immutable Artifacts**: Build once, deploy everywhere
2. **Fast Feedback**: Run tests in parallel
3. **Stages**: Separate building, testing, deployment
4. **Automation**: Minimal manual steps
5. **Rollback**: Easy revert to previous version

---

## 🚀 Part 3: Complete Pipeline Examples

### Example 1: Simple Node.js Pipeline

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Fetching code...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing npm dependencies...'
                sh 'npm install'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'npm run build'
            }
        }
        
        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
        }
        
        stage('Code Quality') {
            steps {
                echo 'Running code analysis...'
                sh 'npm run lint'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        
        stage('Push to Registry') {
            steps {
                echo 'Pushing image to registry...'
                sh '''
                    docker tag myapp:${BUILD_NUMBER} username/myapp:latest
                    docker push username/myapp:latest
                '''
            }
        }
        
        stage('Deploy to Dev') {
            steps {
                echo 'Deploying to development...'
                sh './scripts/deploy-dev.sh'
            }
        }
        
        stage('Integration Tests') {
            steps {
                echo 'Running integration tests...'
                sh 'npm run integration-test'
            }
        }
        
        stage('Approval') {
            steps {
                input 'Deploy to Production?'
            }
        }
        
        stage('Deploy to Production') {
            steps {
                echo 'Deploying to production...'
                sh './scripts/deploy-prod.sh'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
            mail to: 'team@example.com',
                 subject: "Build ${BUILD_NUMBER} succeeded",
                 body: "Build succeeded: ${BUILD_URL}"
        }
        failure {
            echo 'Pipeline failed!'
            mail to: 'team@example.com',
                 subject: "Build ${BUILD_NUMBER} failed",
                 body: "Build failed: ${BUILD_URL}"
        }
    }
}
```

### Example 2: Docker Application Pipeline

**Dockerfile:**
```dockerfile
FROM node:16-alpine as builder

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production

FROM node:16-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules

COPY . .

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js

EXPOSE 3000

CMD ["node", "index.js"]
```

**Jenkinsfile:**
```groovy
pipeline {
    agent any
    
    environment {
        REGISTRY = credentials('docker-registry')
        IMAGE_NAME = "myapp"
        BUILD_DATE = sh(script: "date -u +'%Y-%m-%dT%H:%M:%SZ'", returnStdout: true).trim()
        GIT_COMMIT_SHORT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        
        stage('Build Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        --label "build.date=${BUILD_DATE}" \
                        --label "vcs.ref=${GIT_COMMIT_SHORT}" \
                        .
                '''
            }
        }
        
        stage('Test Image') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:${BUILD_NUMBER} npm test
                    docker run --rm ${IMAGE_NAME}:${BUILD_NUMBER} node healthcheck.js
                '''
            }
        }
        
        stage('Push to Registry') {
            steps {
                sh '''
                    echo $REGISTRY_PSW | docker login -u $REGISTRY_USR --password-stdin
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} $REGISTRY_USR/${IMAGE_NAME}:${BUILD_NUMBER}
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} $REGISTRY_USR/${IMAGE_NAME}:latest
                    docker push $REGISTRY_USR/${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push $REGISTRY_USR/${IMAGE_NAME}:latest
                '''
            }
        }
        
        stage('Deploy Dev') {
            when {
                branch 'develop'
            }
            steps {
                sh './scripts/deploy.sh dev $REGISTRY_USR/${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
        
        stage('Deploy Staging') {
            when {
                branch 'main'
            }
            steps {
                sh './scripts/deploy.sh staging $REGISTRY_USR/${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
        
        stage('Manual Approval') {
            when {
                branch 'main'
            }
            steps {
                input 'Deploy to Production?'
            }
        }
        
        stage('Deploy Production') {
            when {
                branch 'main'
            }
            steps {
                sh './scripts/deploy.sh production $REGISTRY_USR/${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
        }
        failure {
            emailext(
                subject: "Pipeline failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed. See logs at: ${env.BUILD_URL}",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
```

---

## 🧪 Part 4: Testing in Pipelines

### Unit Tests
```groovy
stage('Unit Tests') {
    steps {
        sh 'npm test -- --coverage'
        publishHTML target: [
            reportDir: 'coverage',
            reportFiles: 'index.html',
            reportName: 'Coverage Report'
        ]
    }
}
```

### Integration Tests
```groovy
stage('Integration Tests') {
    steps {
        sh '''
            docker-compose up -d
            sleep 5
            npm run integration-test
            docker-compose down
        '''
    }
}
```

### Security Scanning
```groovy
stage('Security Scan') {
    steps {
        sh 'npm audit'
        sh 'docker scan ${IMAGE_NAME}:${BUILD_NUMBER}'
        sh 'trivy image ${IMAGE_NAME}:${BUILD_NUMBER}'
    }
}
```

### Performance Tests
```groovy
stage('Performance Tests') {
    steps {
        sh '''
            docker run -d -p 3000:3000 --name app ${IMAGE_NAME}:${BUILD_NUMBER}
            sleep 5
            npm run performance-test
            docker stop app
        '''
    }
}
```

---

## 🚀 Part 5: Deployment Strategies

### Blue-Green Deployment
```groovy
stage('Deploy') {
    steps {
        sh '''
            # Deploy to green environment
            docker pull ${IMAGE_NAME}:${BUILD_NUMBER}
            docker-compose -f docker-compose.green.yml up -d
            
            # Run smoke tests
            ./scripts/smoke-test.sh green
            
            # Switch traffic to green
            docker-compose -f docker-compose.nginx.yml config > /tmp/nginx-new.conf
            # Update load balancer
            
            # Keep blue as fallback
        '''
    }
}
```

### Canary Deployment
```groovy
stage('Deploy Canary') {
    steps {
        sh '''
            # Deploy to 10% of traffic
            kubectl set image deployment/myapp \
                myapp=${IMAGE_NAME}:${BUILD_NUMBER} \
                --record \
                --replicas=1
            
            # Monitor metrics
            sleep 300
            
            # Check error rates
            if ./scripts/check-metrics.sh green; then
                kubectl patch deployment myapp -p \
                    '{"spec":{"replicas":10}}'
            fi
        '''
    }
}
```

### Rolling Deployment
```groovy
stage('Rolling Deploy') {
    steps {
        sh '''
            kubectl rolling-update myapp \
                --image=${IMAGE_NAME}:${BUILD_NUMBER} \
                --update-period=30s
        '''
    }
}
```

---

## 📊 Part 6: Monitoring and Logging

### Health Checks
```groovy
stage('Health Check') {
    steps {
        sh '''
            for i in {1..30}; do
                if curl -f http://localhost:3000/health; then
                    echo "Health check passed"
                    exit 0
                fi
                sleep 1
            done
            echo "Health check failed"
            exit 1
        '''
    }
}
```

### Metrics Collection
```groovy
post {
    always {
        sh '''
            docker stats --no-stream > metrics.txt
            curl http://localhost:9090/api/v1/query?query=up > prometheus-metrics.txt
        '''
        archiveArtifacts artifacts: '**/*.txt'
    }
}
```

---

## 🎯 Part 7: Hands-On Exercise

### Step 1: Create GitHub Repository
```bash
git init my-app
cd my-app
echo "node_modules" > .gitignore
```

### Step 2: Create Application
```javascript
// index.js
const express = require('express');
const app = express();

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(3000, () => {
  console.log('App running on port 3000');
});
```

### Step 3: Create Dockerfile
```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "index.js"]
```

### Step 4: Create Jenkinsfile
(Use examples above)

### Step 5: Create GitHub Webhook
1. GitHub Repo → Settings → Webhooks
2. Add webhook
3. Payload URL: `http://jenkins:8080/github-webhook/`
4. Trigger on push

### Step 6: Test Pipeline
```bash
git push origin main
# Jenkins automatically triggers build
```

---

## 📋 Pipeline Best Practices

### 1. Parallel Stages
```groovy
parallel {
    stage('Unit Tests') {
        steps { sh 'npm test' }
    }
    stage('Lint') {
        steps { sh 'npm run lint' }
    }
    stage('Build') {
        steps { sh 'npm run build' }
    }
}
```

### 2. Error Handling
```groovy
stage('Deploy') {
    steps {
        script {
            try {
                sh './deploy.sh'
            } catch (Exception e) {
                echo 'Deployment failed, rolling back...'
                sh './rollback.sh'
                throw e
            }
        }
    }
}
```

### 3. Notifications
```groovy
post {
    always {
        mail to: 'team@example.com',
             subject: "Build ${BUILD_NUMBER}",
             body: "Status: ${currentBuild.result}"
    }
}
```

### 4. Artifact Management
```groovy
archiveArtifacts artifacts: '**/*.jar,**/*.war'
cleanWs() // Clean workspace
```

---

## ✅ Checklist

- [ ] Understand CI/CD pipeline stages
- [ ] Know different deployment strategies
- [ ] Can write complete Jenkinsfile
- [ ] Set up GitHub integration
- [ ] Configure webhooks
- [ ] Implement testing stages
- [ ] Set up artifact storage
- [ ] Configure notifications
- [ ] Monitor pipeline health
- [ ] Practice parallel execution
- [ ] Know error handling and rollback

