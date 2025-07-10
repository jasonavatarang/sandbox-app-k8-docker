# sandbox-app-k8-docker

# Java + React + Docker + Kubernetes Sample Project

This README will walk you through setting up a full-stack Java (Spring Boot) + React application using Docker and Kubernetes.

---

## Prerequisites

### Install Tools
1. **Docker**
   - Download: https://www.docker.com/products/docker-desktop
   - Verify: `docker --version`

2. **Kubernetes (via Docker Desktop or minikube)**
   - Docker Desktop includes Kubernetes (Enable via Settings)
   - Or install minikube: https://minikube.sigs.k8s.io/docs/start/
   - Verify: `kubectl version --client`

3. **Java SDK (17 recommended)**
   - Download: https://adoptium.net/en-GB/temurin/releases/
   - Verify: `java -version`

4. **Maven**
   - Download: https://maven.apache.org/download.cgi
   - Verify: `mvn -v`

5. **Node.js + npm**
   - Download: https://nodejs.org/
   - Verify: `node -v` and `npm -v`

6. **Yarn (Optional)**
   - Install: `npm install --global yarn`

---

## Folder Structure

```
project-root/
├── backend/                # Java Spring Boot backend
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── frontend/               # React app
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── k8s/                    # Kubernetes manifests
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── services.yaml
└── README.md
```

---

## Steps to Create Project From Scratch

### 1. Backend (Java + Spring Boot)
```bash
mkdir backend && cd backend
curl https://start.spring.io/starter.zip \
  -d dependencies=web \
  -d type=maven-project \
  -d language=java \
  -d name=backend \
  -d javaVersion=17 \
  -o backend.zip
unzip backend.zip && rm backend.zip
```
- Add a REST controller under `src/main/java/.../controller/HelloController.java`
```java
@RestController
public class HelloController {
    @GetMapping("/api/hello")
    public String hello() {
        return "Hello from backend!";
    }
}
```

### 2. Frontend (React)
```bash
npx create-react-app frontend
cd frontend
npm install axios
```
- Add Axios call to `/api/hello` and display response in component.

### 3. Dockerize
**Backend Dockerfile:**
```Dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY . .
RUN ./mvnw package
CMD ["java", "-jar", "target/*.jar"]
```

**Frontend Dockerfile:**
```Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install && npm run build
RUN npm install -g serve
CMD ["serve", "-s", "build"]
```

### 4. Kubernetes Manifests
**k8s/backend-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: backend-image-name
        ports:
        - containerPort: 8080
```

**k8s/frontend-deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: frontend-image-name
        ports:
        - containerPort: 3000
```

**k8s/services.yaml:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

---

## Running the App

### 1. Build Docker Images
```bash
# From backend folder
docker build -t backend-image-name .
# From frontend folder
docker build -t frontend-image-name .
```

### 2. Start Kubernetes Cluster (Minikube or Docker Desktop)
```bash
minikube start
kubectl apply -f k8s/
minikube service frontend-service
```


