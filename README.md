# ☸️ Project 5: Three-Tier Application Deployment on AWS EKS

## 📌 Project Overview

This project demonstrates the deployment of a **three-tier application** on **AWS EKS** (Kubernetes).
The three tiers include:

1. **Frontend** – User interface (web server)
2. **Backend** – Application server (Node.js / Django)
3. **Database** – Persistent storage (MySQL/Postgres)

By using **Kubernetes**, the project ensures:

* Scalability
* High availability
* Automated deployment and rollback
* Self-healing applications

---

## 🛠️ Tech Stack / Tools Required

* **AWS EKS** (Kubernetes managed service)
* **kubectl** (Kubernetes CLI)
* **Docker & Docker Hub / ECR** (Container images)
* **Node.js / Django** (Application backend)
* **MySQL / PostgreSQL** (Database)
* **Helm** (Optional – Kubernetes package management)
* **Git & GitHub** (Version control)

---

## 📂 Functional Requirements

* Deploy frontend, backend, and database on separate pods.
* Enable communication between pods using **Kubernetes Services**.
* Store database data persistently using **Persistent Volumes**.
* Auto-scale pods based on load.
* Monitor pod health and restart automatically on failure.

---

## 📂 Non-Functional Requirements

* Ensure **high availability** using multiple replicas.
* Maintain **secure communication** between pods and database.
* Use **Docker images** stored in a registry for deployment.
* Enable **rolling updates** for zero-downtime deployments.

---

## 🔄 Project Flow

1. **Dockerize each tier**

   * Frontend (web)
   * Backend (application server)
   * Database (MySQL/Postgres)

2. **Push images** to Docker Hub or AWS ECR.

3. **Kubernetes manifests**:

   * Deployments: Define pods and replicas
   * Services: Expose frontend and backend
   * PersistentVolume & PersistentVolumeClaim: Database storage

4. **Deploy on AWS EKS** using `kubectl apply -f <manifest>`

5. **Access application**

   * Frontend exposed via LoadBalancer
   * Backend communicates internally with database
   * Database persists data in PV

6. **Monitoring & Scaling**

   * Use `kubectl get pods`, `kubectl get svc` to monitor
   * Auto-scale backend pods if required (`kubectl autoscale deployment`)

---

## 📖 Folder Structure

```
project-5-aws-eks/
│-- README.md
│-- docker/
│   ├── frontend/
│   │   ├── Dockerfile
│   │   └── source files
│   ├── backend/
│   │   ├── Dockerfile
│   │   └── source files
│   └── database/
│       └── init.sql
│-- k8s/
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── db-deployment.yaml
│   ├── services.yaml
│   ├── pv-pvc.yaml
│-- .gitignore
```

---

## 📝 Sample `frontend-deployment.yaml`

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
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

> Similar manifests are used for backend and database.

---

## 📝 Resume Highlights

**Objective**
Deployed a fully containerized three-tier application on **AWS EKS**, enabling scalability, availability, and automated management.

**Key Achievements**

* Dockerized frontend, backend, and database layers.
* Configured **Kubernetes Deployments, Services, and Persistent Volumes**.
* Automated deployment using **kubectl and Helm charts**.

**Impact**
✅ Achieved zero-downtime deployment with rolling updates.
✅ Improved scalability and availability of application pods.
✅ Ensured persistent storage and data integrity for database.

---

## 🔗 Repository Link

```md
https://github.com/<your-username>/project-5-aws-eks
```

