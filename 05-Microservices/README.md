# Practical 5: Introduction to Microservices

## 📋 Overview
Understand microservices architecture and how Docker enables microservices-based applications.

---

## 🏛️ Part 1: Microservices Fundamentals

### What are Microservices?
Microservices architecture breaks down applications into small, independent services that communicate via APIs.

### Monolithic vs Microservices

#### Monolithic Architecture
```
┌─────────────────────────────┐
│     Single Application      │
├─────────────────────────────┤
│ User Service                │
│ Product Service             │
│ Order Service               │
│ Payment Service             │
└─────────────────────────────┘
```

Problems:
- Hard to scale individual services
- One failure affects entire app
- Difficult to update without full deployment
- Technology lock-in

#### Microservices Architecture
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ User Service │  │Product Service│ │Order Service │  │PaymentService│
├──────────────┤  ├──────────────┤  ├──────────────┤  ├──────────────┤
│ Node.js      │  │Python/Django │ │Java/Spring   │  │Go            │
│ Port: 3001   │  │Port: 5001    │  │Port: 8001    │  │Port: 8080    │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
       ↑                 ↑                   ↑                ↑
       └─────────────────┴───────────────────┴────────────────┘
                    API Gateway (Port: 8000)
```

Advantages:
- Independent scaling
- Fault isolation
- Technology flexibility
- Faster deployment
- Easier testing

---

## 🐳 Part 2: Microservices with Docker

### Why Docker for Microservices?
1. **Consistency**: Same environment in dev/test/prod
2. **Isolation**: Each service in its own container
3. **Easy Deployment**: Simple versioning and rollback
4. **Resource Efficiency**: Lightweight containers
5. **Orchestration**: Easy to manage multiple services

### Service Structure

```
my-app/
├── api-gateway/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
├── user-service/
│   ├── Dockerfile
│   ├── package.json
│   └── index.js
├── product-service/
│   ├── Dockerfile
│   ├── go.mod
│   └── main.go
├── docker-compose.yml
└── README.md
```

---

## 🛠️ Part 3: Building Microservices

### Example 1: API Gateway (Python)

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

**requirements.txt:**
```
flask==2.3.0
requests==2.31.0
```

**app.py:**
```python
from flask import Flask, jsonify
import requests

app = Flask(__name__)

USER_SERVICE = "http://user-service:3001"
PRODUCT_SERVICE = "http://product-service:5001"
ORDER_SERVICE = "http://order-service:8001"

@app.route('/api/users/<user_id>', methods=['GET'])
def get_user(user_id):
    response = requests.get(f"{USER_SERVICE}/users/{user_id}")
    return jsonify(response.json())

@app.route('/api/products', methods=['GET'])
def get_products():
    response = requests.get(f"{PRODUCT_SERVICE}/products")
    return jsonify(response.json())

@app.route('/api/orders', methods=['GET'])
def get_orders():
    response = requests.get(f"{ORDER_SERVICE}/orders")
    return jsonify(response.json())

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)
```

### Example 2: User Service (Node.js)

**Dockerfile:**
```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package.json .
RUN npm install

COPY . .

EXPOSE 3001

CMD ["node", "index.js"]
```

**package.json:**
```json
{
  "name": "user-service",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "body-parser": "^1.20.0"
  }
}
```

**index.js:**
```javascript
const express = require('express');
const app = express();

app.use(express.json());

const users = {
  '1': { id: '1', name: 'John', email: 'john@example.com' },
  '2': { id: '2', name: 'Jane', email: 'jane@example.com' }
};

app.get('/users/:id', (req, res) => {
  const user = users[req.params.id];
  if (user) {
    res.json(user);
  } else {
    res.status(404).json({ error: 'User not found' });
  }
});

app.get('/users', (req, res) => {
  res.json(Object.values(users));
});

app.listen(3001, () => {
  console.log('User service running on port 3001');
});
```

### Example 3: Product Service (Go)

**Dockerfile:**
```dockerfile
FROM golang:1.18-alpine as builder

WORKDIR /app

COPY go.mod .
COPY main.go .

RUN go build -o product-service .

FROM alpine:latest

WORKDIR /app

COPY --from=builder /app/product-service .

EXPOSE 5001

CMD ["./product-service"]
```

**main.go:**
```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
)

type Product struct {
    ID    string  `json:"id"`
    Name  string  `json:"name"`
    Price float64 `json:"price"`
}

var products = []Product{
    {ID: "1", Name: "Laptop", Price: 999.99},
    {ID: "2", Name: "Phone", Price: 699.99},
}

func getProducts(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(products)
}

func getProduct(w http.ResponseWriter, r *http.Request) {
    id := strings.TrimPrefix(r.URL.Path, "/products/")
    w.Header().Set("Content-Type", "application/json")
    
    for _, p := range products {
        if p.ID == id {
            json.NewEncoder(w).Encode(p)
            return
        }
    }
    w.WriteHeader(http.StatusNotFound)
    json.NewEncoder(w).Encode(map[string]string{"error": "Product not found"})
}

func main() {
    http.HandleFunc("/products", getProducts)
    http.HandleFunc("/products/", getProduct)
    
    fmt.Println("Product service running on port 5001")
    log.Fatal(http.ListenAndServe(":5001", nil))
}
```

---

## 🐳 Part 4: Docker Compose for Microservices

**docker-compose.yml:**
```yaml
version: '3.9'

services:
  api-gateway:
    build:
      context: ./api-gateway
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    depends_on:
      - user-service
      - product-service
      - order-service
    networks:
      - microservices
    environment:
      - FLASK_ENV=development

  user-service:
    build:
      context: ./user-service
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    networks:
      - microservices
    environment:
      - NODE_ENV=production

  product-service:
    build:
      context: ./product-service
      dockerfile: Dockerfile
    ports:
      - "5001:5001"
    networks:
      - microservices

  order-service:
    build:
      context: ./order-service
      dockerfile: Dockerfile
    ports:
      - "8001:8001"
    networks:
      - microservices

  database:
    image: postgres:13
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: password
      POSTGRES_DB: microservices_db
    ports:
      - "5432:5432"
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - microservices

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - microservices

networks:
  microservices:
    driver: bridge

volumes:
  db-data:
```

---

## 🚀 Part 5: Communication Patterns

### 1. Synchronous (Request/Response)
```python
# API Gateway calls User Service
response = requests.get('http://user-service:3001/users/1')
user_data = response.json()
```

### 2. Asynchronous (Message Queue)
```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
```

### 3. Event-Driven
```python
# Service publishes event
events.publish('user.created', {'user_id': 123})

# Other services subscribe
@events.subscribe('user.created')
def on_user_created(event):
    send_welcome_email(event['user_id'])
```

---

## 🎯 Hands-On Exercise: Complete Microservices Stack

### Step 1: Create Project Structure
```bash
mkdir microservices-app && cd microservices-app

mkdir api-gateway user-service product-service order-service
```

### Step 2: Create Services (follow examples above)

### Step 3: Create docker-compose.yml (see above)

### Step 4: Deploy
```bash
# Build and start all services
docker-compose up -d

# Verify services
docker-compose ps

# Check logs
docker-compose logs -f

# Call API
curl http://localhost:8000/api/users/1
curl http://localhost:8000/api/products

# Stop services
docker-compose down
```

---

## 📊 Microservices Best Practices

### 1. Database per Service
```yaml
services:
  user-service:
    depends_on:
      - user-db
  
  user-db:
    image: postgres:13
```

### 2. Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1
```

### 3. Logging and Monitoring
```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.route('/api/users')
def get_users():
    logger.info("Fetching users")
    # ...
```

### 4. Service Discovery
- Kubernetes DNS
- Consul
- Eureka
- Docker DNS (built-in)

### 5. API Versioning
```python
@app.route('/api/v1/users')
def get_users_v1():
    # ...

@app.route('/api/v2/users')
def get_users_v2():
    # Enhanced version
```

---

## ✅ Checklist

- [ ] Understand microservices architecture
- [ ] Know monolithic vs microservices trade-offs
- [ ] Can create multiple services with Docker
- [ ] Use Docker Compose to manage services
- [ ] Services communicate via network
- [ ] Understand inter-service communication patterns
- [ ] Familiar with service discovery
- [ ] Can scale individual services
- [ ] Know when to use microservices

