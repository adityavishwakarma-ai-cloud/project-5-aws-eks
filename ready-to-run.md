## 1️⃣ Frontend – `docker/frontend/Dockerfile`

```dockerfile
# Frontend Dockerfile
FROM nginx:alpine

# Remove default nginx website
RUN rm -rf /usr/share/nginx/html/*

# Copy frontend files
COPY ./ /usr/share/nginx/html

# Expose port
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

---

### Sample Frontend HTML – `docker/frontend/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>Project 5 - Frontend</title>
</head>
<body>
    <h1>Welcome to Project 5: Three-Tier App</h1>
    <p>Frontend is running!</p>
</body>
</html>
```

---

## 2️⃣ Backend – `docker/backend/Dockerfile`

```dockerfile
# Backend Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy backend source code
COPY . .

# Expose port
EXPOSE 3000

# Start backend server
CMD ["node", "app.js"]
```

---

### Sample Backend – `docker/backend/app.js`

```javascript
const express = require('express');
const mysql = require('mysql');
const app = express();
const port = 3000;

// Database connection
const db = mysql.createConnection({
  host: 'db-service',   // matches k8s service name
  user: 'appuser',
  password: 'app123',
  database: 'appdb'
});

db.connect(err => {
  if (err) console.error('Database connection failed:', err);
  else console.log('Connected to MySQL database');
});

// Sample API
app.get('/api/users', (req, res) => {
  db.query('SELECT * FROM users', (err, results) => {
    if (err) res.send('Error fetching users');
    else res.json(results);
  });
});

app.listen(port, () => {
  console.log(`Backend running at http://localhost:${port}`);
});
```

---

### `docker/backend/package.json`

```json
{
  "name": "three-tier-backend",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mysql": "^2.18.1"
  }
}
```

---

## 3️⃣ Database – `docker/database/init.sql`

```sql
CREATE DATABASE IF NOT EXISTS appdb;

USE appdb;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(50)
);

INSERT INTO users (name, email) VALUES
('Alice','alice@example.com'),
('Bob','bob@example.com'),
('Charlie','charlie@example.com');
```

---

## 4️⃣ Kubernetes Manifests – `k8s/`

### a) `k8s/db-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root
        - name: MYSQL_DATABASE
          value: appdb
        - name: MYSQL_USER
          value: appuser
        - name: MYSQL_PASSWORD
          value: app123
        volumeMounts:
        - name: db-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: db-storage
        persistentVolumeClaim:
          claimName: db-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: db
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  type: ClusterIP
```

---

### b) `k8s/backend-deployment.yaml`

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
        image: <dockerhub-username>/backend:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
  type: ClusterIP
```

---

### c) `k8s/frontend-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
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
        image: <dockerhub-username>/frontend:latest
        ports:
        - containerPort: 80
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
      targetPort: 80
  type: LoadBalancer
```

---

### d) `k8s/pv-pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/db"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 5️⃣ Folder Structure

```
project-5-aws-eks/
│-- README.md
│-- docker/
│   ├── frontend/
│   │   ├── Dockerfile
│   │   └── index.html
│   ├── backend/
│   │   ├── Dockerfile
│   │   ├── app.js
│   │   └── package.json
│   └── database/
│       └── init.sql
│-- k8s/
│   ├── db-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── pv-pvc.yaml
│-- .gitignore
```

---

## ⚡ How to Deploy on AWS EKS

1. **Build Docker images and push to Docker Hub or ECR**

```bash
docker build -t <username>/frontend:latest docker/frontend
docker build -t <username>/backend:latest docker/backend
docker push <username>/frontend:latest
docker push <username>/backend:latest
```

2. **Apply Kubernetes manifests**

```bash
kubectl apply -f k8s/pv-pvc.yaml
kubectl apply -f k8s/db-deployment.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
```

3. **Check pods and services**

```bash
kubectl get pods
kubectl get svc
```

4. **Access frontend** via LoadBalancer external IP.

---
