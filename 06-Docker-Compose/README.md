# Practical 6: Docker Compose - Installation, YAML, & WordPress Deployment

## 📋 Overview
Master Docker Compose for orchestrating multi-container applications with YAML configuration.

---

## 📦 Part 1: Docker Compose Installation

### What is Docker Compose?
A tool for defining and running multi-container Docker applications using YAML files.

### Installation

#### Windows & Mac
```bash
# Docker Desktop includes Docker Compose
docker-compose --version

# Or update to latest
docker-compose up --upgrade
```

#### Linux
```bash
# Install Docker first
sudo apt-get install docker.io

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker-compose --version
```

#### Using Python pip
```bash
pip install docker-compose
```

---

## 📝 Part 2: Understanding YAML

### YAML Basics
YAML is human-readable data serialization language. Key concepts:

#### Syntax
```yaml
# Comments start with #

# Key-value pairs
key: value
name: "John"
age: 30

# Lists
items:
  - apple
  - banana
  - orange

# Or inline list
fruits: [apple, banana, orange]

# Nested objects
person:
  name: John
  age: 30
  email: john@example.com

# Strings with special characters
description: "This is a
  multi-line
  string"

# Environment variables
env_var: ${MY_ENV_VAR}
```

#### Data Types
```yaml
string: "hello"
number: 42
float: 3.14
boolean: true
null_value: null
list: [1, 2, 3]
object: {key: value}
```

---

## 🐳 Part 3: Docker Compose File Structure

### Basic Structure
```yaml
version: '3.9'  # Compose file format version

services:       # Container definitions
  web:
    image: nginx
  db:
    image: postgres

networks:       # Custom networks
  app-network:

volumes:        # Named volumes
  db-data:
```

### Complete Example
```yaml
version: '3.9'

services:
  # Web Service
  web:
    image: nginx:latest
    container_name: web-server
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - web-data:/usr/share/nginx/html
    depends_on:
      - app
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Application Service
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
    container_name: app-server
    ports:
      - "3000:3000"
    environment:
      DB_HOST: postgres-db
      DB_PORT: 5432
      REDIS_URL: redis://redis-cache:6379
    volumes:
      - ./app:/app
      - /app/node_modules
    depends_on:
      - postgres-db
      - redis-cache
    networks:
      - app-network
    restart: always

  # Database Service
  postgres-db:
    image: postgres:14
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secretpassword
      POSTGRES_DB: myapp_db
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache Service
  redis-cache:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network
    command: redis-server --appendonly yes

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
  web-data:
```

---

## 📋 Part 4: Common Compose Commands

### Basic Commands
```bash
# Build services
docker-compose build

# Start services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# List running services
docker-compose ps

# View logs
docker-compose logs

# Follow logs (real-time)
docker-compose logs -f

# View logs for specific service
docker-compose logs web

# Execute command in service
docker-compose exec web bash

# Restart services
docker-compose restart

# Scale service
docker-compose up -d --scale worker=3
```

### Advanced Commands
```bash
# Pull images
docker-compose pull

# Push images (if using custom registry)
docker-compose push

# Remove stopped containers
docker-compose rm

# Validate compose file
docker-compose config

# Show environment variables
docker-compose env

# Remove images
docker-compose down --rmi all

# View network
docker-compose network ls
```

---

## 🎯 Part 5: WordPress with Docker Compose

WordPress is a popular CMS. Here's how to deploy it with Docker Compose.

### WordPress Stack Components
- **WordPress**: Web server and CMS
- **MySQL**: Database
- **PhpMyAdmin**: Database management interface
- **Volumes**: Persistent storage

### Docker Compose File

**docker-compose.yml:**
```yaml
version: '3.9'

services:
  # MySQL Database
  mysql:
    image: mysql:8.0
    container_name: wordpress-db
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress_password
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - wordpress-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # WordPress
  wordpress:
    image: wordpress:latest
    container_name: wordpress-app
    depends_on:
      - mysql
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress_password
      WORDPRESS_TABLE_PREFIX: wp_
    volumes:
      - wordpress-data:/var/www/html
    networks:
      - wordpress-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

  # PhpMyAdmin (optional - for database management)
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wordpress-phpmyadmin
    depends_on:
      - mysql
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql
      PMA_USER: wordpress
      PMA_PASSWORD: wordpress_password
    networks:
      - wordpress-network
    restart: unless-stopped

networks:
  wordpress-network:
    driver: bridge

volumes:
  mysql-data:
    driver: local
  wordpress-data:
    driver: local
```

### Step-by-Step Deployment

#### Step 1: Create Directory and File
```bash
mkdir wordpress-docker && cd wordpress-docker

# Copy the docker-compose.yml above into this directory
```

#### Step 2: Start Services
```bash
# Build and start all services
docker-compose up -d

# Verify services are running
docker-compose ps

# Check logs
docker-compose logs -f wordpress
```

#### Step 3: Access WordPress
```
WordPress: http://localhost:8080
PhpMyAdmin: http://localhost:8081
```

#### Step 4: Initial Setup
1. Open browser to `http://localhost:8080`
2. Select language
3. Fill in site information:
   - Site Title: "My Blog"
   - Username: "admin"
   - Password: Choose strong password
   - Email: Your email
4. Install WordPress

#### Step 5: Manage Services
```bash
# View logs
docker-compose logs wordpress

# Execute command in WordPress container
docker-compose exec wordpress wp db info

# Stop services
docker-compose stop

# Start services
docker-compose start

# Restart all services
docker-compose restart

# Stop and remove containers (keep volumes)
docker-compose down

# Remove everything including volumes (careful!)
docker-compose down -v
```

---

## 🔧 Part 6: Advanced Docker Compose

### Environment Variables
```yaml
# .env file
MYSQL_ROOT_PASSWORD=root123
WORDPRESS_DB_PASSWORD=wp123
```

**docker-compose.yml:**
```yaml
services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

### Multiple Compose Files
```bash
# Merge compose files
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Override Configuration
**docker-compose.override.yml:**
```yaml
services:
  wordpress:
    ports:
      - "9000:80"
    environment:
      WORDPRESS_DEBUG: "true"
```

### Build from Dockerfile
```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        NODE_VERSION: 16
    image: myapp:1.0
```

---

## 🎓 Hands-On Exercises

### Exercise 1: Simple Web App
Create a compose file with:
- Nginx web server
- Node.js application
- MongoDB database

**Solution:**
```yaml
version: '3.9'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"

  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    ports:
      - "3000:3000"

  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
```

### Exercise 2: WordPress Deployment
Follow the WordPress section above.

### Exercise 3: Add Monitoring
Add Prometheus and Grafana to the WordPress stack.

### Exercise 4: Multi-Environment
Create:
- docker-compose.yml (base)
- docker-compose.dev.yml (development overrides)
- docker-compose.prod.yml (production overrides)

---

## 📊 Compose File Reference

| Service Key | Purpose |
|-------------|---------|
| image | Image to use |
| build | Build configuration |
| ports | Port mapping |
| volumes | Volume/mount configuration |
| environment | Environment variables |
| depends_on | Service dependencies |
| networks | Network configuration |
| restart | Restart policy |
| healthcheck | Health check configuration |
| container_name | Container name |

---

## 🔍 Troubleshooting

### Services won't start
```bash
# Check logs
docker-compose logs

# Check service status
docker-compose ps

# Rebuild images
docker-compose build --no-cache
```

### Can't access services
```bash
# Check port mappings
docker-compose ps

# Test connectivity
docker-compose exec web curl http://db:5432
```

### Volumes not working
```bash
# Check volume mounts
docker-compose exec web mount | grep /data

# Check volume contents
docker volume inspect volume_name
```

---

## ✅ Checklist

- [ ] Docker Compose installed
- [ ] Understand YAML syntax
- [ ] Know compose file structure
- [ ] Familiar with common commands
- [ ] Can create multi-service applications
- [ ] Deployed WordPress successfully
- [ ] Set up volumes for persistence
- [ ] Configured networking between services
- [ ] Use environment variables
- [ ] Know restart policies

