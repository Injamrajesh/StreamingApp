# StreamingApp – End-to-End DevOps, CI/CD, Kubernetes & Scaling

## 1. Project Overview

**StreamingApp** is a MERN-based streaming application deployed using modern DevOps and cloud-native technologies.

This project demonstrates an end-to-end DevOps workflow covering:

* Source code management with GitHub
* Docker containerization
* Amazon Elastic Container Registry (ECR)
* Jenkins CI/CD automation
* Amazon Elastic Kubernetes Service (EKS)
* Kubernetes deployments and services
* Helm-based application deployment
* Horizontal Pod Autoscaling (HPA)
* MongoDB persistent storage
* AWS EBS CSI Driver
* Amazon CloudWatch monitoring
* Centralized Kubernetes logging
* CloudWatch CPU alarms
* Application health and scalability validation

The objective is to build, deploy, monitor, and scale a containerized MERN application on AWS.

---

# 2. Project Objectives

The main objectives of this project are:

1. Containerize all application components using Docker.
2. Create dedicated Amazon ECR repositories for each component.
3. Automate Docker image build and push using Jenkins.
4. Provision an Amazon EKS Kubernetes cluster.
5. Package the application using Helm.
6. Deploy the complete application to EKS.
7. Configure Kubernetes services for application communication.
8. Implement Horizontal Pod Autoscaling.
9. Configure persistent storage for MongoDB.
10. Implement centralized monitoring and logging using CloudWatch.
11. Configure CloudWatch alarms for cluster CPU utilization.
12. Validate application accessibility and scalability.

---

# 3. Architecture

```text
                           ┌────────────────────┐
                           │      GitHub        │
                           │  StreamingApp Repo │
                           └─────────┬──────────┘
                                     │
                                     ▼
                           ┌────────────────────┐
                           │      Jenkins       │
                           │   CI/CD Pipeline   │
                           └─────────┬──────────┘
                                     │
                         Build / Test / Push
                                     │
                                     ▼
                           ┌────────────────────┐
                           │    Amazon ECR      │
                           ├────────────────────┤
                           │ Frontend           │
                           │ Auth Service       │
                           │ Streaming Service  │
                           │ Admin Service      │
                           │ Chat Service       │
                           └─────────┬──────────┘
                                     │
                                     ▼
                    ┌────────────────────────────────┐
                    │       Amazon EKS Cluster       │
                    │        streamingapp-eks        │
                    │                                │
                    │  ┌──────────────────────────┐  │
                    │  │ Frontend ×2              │  │
                    │  │ Auth ×2                  │  │
                    │  │ Streaming ×2             │  │
                    │  │ Admin ×2                 │  │
                    │  │ Chat ×2                  │  │
                    │  │ MongoDB ×1               │  │
                    │  └──────────────────────────┘  │
                    │                                │
                    │        Helm + Kubernetes       │
                    │        HPA: 2–5 replicas       │
                    └───────────────┬────────────────┘
                                    │
                                    ▼
                           ┌─────────────────┐
                           │ AWS LoadBalancer│
                           └────────┬────────┘
                                    │
                                    ▼
                              End User
                               Browser

                    ┌────────────────────────────┐
                    │ Amazon CloudWatch          │
                    │                            │
                    │ • Container Insights       │
                    │ • Metrics                  │
                    │ • Application Logs         │
                    │ • Host Logs                │
                    │ • Dataplane Logs           │
                    │ • Performance Logs          │
                    │ • CPU Alarm                │
                    └────────────────────────────┘
```

---

# 4. Technology Stack

| Technology         | Purpose                               |
| ------------------ | ------------------------------------- |
| GitHub             | Source code management                |
| Git                | Version control                       |
| Docker             | Application containerization          |
| Jenkins            | CI/CD automation                      |
| Amazon ECR         | Container image registry              |
| Amazon EKS         | Kubernetes orchestration              |
| Kubernetes         | Container management                  |
| Helm               | Kubernetes application packaging      |
| MongoDB            | Application database                  |
| AWS EBS CSI Driver | Persistent storage                    |
| Amazon CloudWatch  | Monitoring and logging                |
| Container Insights | Kubernetes observability              |
| Node.js            | Backend services                      |
| React              | Frontend                              |
| Nginx              | Frontend web server and reverse proxy |

---

# 5. Repository

GitHub repository:

https://github.com/Injamrajesh/StreamingApp

The repository contains the application source code, Dockerfiles, Jenkins pipeline, Helm chart, and deployment configuration.

---

# 6. Repository Structure

```text
StreamingApp/
│
├── backend/
│   ├── adminService/
│   │   └── Dockerfile
│   │
│   ├── authService/
│   │   └── Dockerfile
│   │
│   ├── chatService/
│   │   └── Dockerfile
│   │
│   └── streamingService/
│       └── Dockerfile
│
├── frontend/
│   ├── public/
│   ├── src/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── .dockerignore
│
├── helm/
│   └── streamingapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
├── docker-compose.yml
├── Jenkinsfile
├── .env.example
├── .gitignore
└── README.md
```

---

# 7. Application Components

The application consists of five Dockerized application components.

| Component         |  Port | Kubernetes Service |
| ----------------- | ----: | ------------------ |
| Frontend          |    80 | LoadBalancer       |
| Auth Service      |  3001 | ClusterIP          |
| Streaming Service |  3002 | ClusterIP          |
| Admin Service     |  3003 | ClusterIP          |
| Chat Service      |  3004 | ClusterIP          |
| MongoDB           | 27017 | ClusterIP          |

---

# 8. Docker Containerization

Each application component has its own Dockerfile.

## Frontend

The frontend uses a multi-stage Docker build:

```text
Node.js build stage
        ↓
npm install
        ↓
React production build
        ↓
Nginx production image
```

The final frontend image runs using Nginx.

The Nginx configuration also provides:

* React SPA routing
* Backend API reverse proxy
* WebSocket support for chat
* Internal Kubernetes service routing

---

# 9. Local Docker Testing

Create the environment file:

```powershell
Copy-Item .env.example .env
```

Build the application:

```powershell
docker compose build
```

Start all services:

```powershell
docker compose up -d
```

Check running containers:

```powershell
docker compose ps
```

Stop the application:

```powershell
docker compose down
```

The application was successfully validated locally before deployment to EKS.

---

# 10. Amazon ECR

Five dedicated ECR repositories were created.

```text
streamingapp-frontend
streamingapp-auth-service
streamingapp-streaming-service
streamingapp-admin-service
streamingapp-chat-service
```

Each application component has its own ECR repository.

Example ECR repository:

```text
310297108115.dkr.ecr.us-east-1.amazonaws.com/streamingapp-frontend
```

Images are tagged using the Git commit SHA.

Example:

```text
e743d87
```

This provides traceability between:

```text
Git Commit
    ↓
Docker Image
    ↓
ECR
    ↓
Kubernetes Deployment
```

---

# 11. ECR Authentication

Authenticate Docker with Amazon ECR:

```powershell
aws ecr get-login-password --region us-east-1 |
docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

---

# 12. Docker Image Build and Push

Example frontend image build:

```powershell
docker build `
  --build-arg REACT_APP_AUTH_API_URL=/api `
  --build-arg REACT_APP_STREAMING_API_URL=/api `
  --build-arg REACT_APP_STREAMING_PUBLIC_URL= `
  --build-arg REACT_APP_ADMIN_API_URL=/api/admin `
  --build-arg REACT_APP_CHAT_API_URL=/api/chat `
  --build-arg REACT_APP_CHAT_SOCKET_URL=/ `
  -t streamingapp-frontend:e743d87 ./frontend
```

Tag the image:

```powershell
docker tag streamingapp-frontend:e743d87 `
  <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/streamingapp-frontend:e743d87
```

Push the image:

```powershell
docker push `
  <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/streamingapp-frontend:e743d87
```

The same process is used for the backend services.

---

# 13. Jenkins CI/CD

Jenkins is used to automate the application build and deployment workflow.

Jenkins:

```text
GitHub
   ↓
Checkout
   ↓
Install Dependencies
   ↓
Run Tests
   ↓
Build Docker Images
   ↓
Authenticate with ECR
   ↓
Push Images to ECR
```

Jenkins job:

```text
StreamingApp-CI-CD-RI
```

The pipeline uses credentials stored securely in Jenkins.

No AWS access keys or passwords are stored directly in the Jenkinsfile.

---

# 14. Jenkins Pipeline Stages

The Jenkinsfile contains stages for:

### Checkout

Retrieves the latest source code from GitHub.

### Install Dependencies

Installs required Node.js dependencies.

### Run Tests

Executes application tests before creating production images.

### Build Docker Images

Builds Docker images for:

```text
Frontend
Auth
Streaming
Admin
Chat
```

### AWS/ECR Login

Authenticates Jenkins with Amazon ECR.

### Push Images

Pushes the Docker images to their corresponding ECR repositories.

Images are tagged using the Git commit SHA.

---

# 15. Amazon EKS

The application is deployed to an Amazon EKS cluster.

Cluster name:

```text
streamingapp-eks
```

AWS Region:

```text
us-east-1
```

Kubernetes version:

```text
1.34
```

Worker nodes:

```text
2 × t3.medium
```

Node group scaling:

```text
Minimum: 2
Desired: 2
Maximum: 3
```

---

# 16. EKS Cluster Provisioning

The cluster was provisioned using `eksctl`.

Example:

```powershell
eksctl create cluster -f eksctl-config.yaml
```

The cluster configuration enables:

* Managed node groups
* OIDC
* Two worker nodes
* Node autoscaling capacity
* EKS managed infrastructure

---

# 17. Kubernetes Storage

MongoDB uses persistent Kubernetes storage.

The AWS EBS CSI Driver was installed and configured.

Storage configuration:

```text
Storage Class: gp2
Size: 10Gi
```

MongoDB uses a PersistentVolumeClaim so that database storage is not tied directly to the lifetime of a MongoDB pod.

---

# 18. Helm Deployment

The application is packaged as a Helm chart.

Chart location:

```text
helm/streamingapp
```

Chart information:

```text
Name: streamingapp
Version: 1.0.0
App Version: 1.0.0
```

Validate the Helm chart:

```powershell
helm lint .\helm\streamingapp
```

Install or upgrade:

```powershell
helm upgrade --install streamingapp `
  .\helm\streamingapp `
  --namespace streamingapp `
  --create-namespace
```

Check Helm deployment:

```powershell
helm list -n streamingapp
```

---

# 19. Kubernetes Services

The application uses Kubernetes service discovery.

```text
Frontend
   │
   ├── /api/              → Auth Service
   ├── /api/streaming/    → Streaming Service
   ├── /api/admin/        → Admin Service
   └── /api/chat/         → Chat Service
```

The frontend is exposed externally through an AWS LoadBalancer.

The backend services remain internal using ClusterIP services.

This prevents direct public exposure of the backend services.

---

# 20. Kubernetes Deployment Status

The final deployment contained:

```text
Frontend       2 replicas
Auth           2 replicas
Streaming      2 replicas
Admin          2 replicas
Chat           2 replicas
MongoDB        1 replica
```

Total:

```text
11 application/database pods
```

All pods were successfully running with:

```text
READY: 1/1
STATUS: Running
RESTARTS: 0
```

The workloads were distributed across both EKS worker nodes.

---

# 21. Horizontal Pod Autoscaling

Horizontal Pod Autoscaler (HPA) was configured for all five application services
