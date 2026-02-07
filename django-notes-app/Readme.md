# Django Notes App – CI/CD with GitLab & Kubernetes

This project demonstrates a production-style CI/CD pipeline for a Django application using Docker, GitLab CI, and Kubernetes (kind). The pipeline builds a Docker image, pushes it to DockerHub, and deploys the application to a Kubernetes cluster using a Deployment and Service.

---

## 📌 High-Level Flow

```
Code Push
   ↓
GitLab CI Pipeline
   ↓
Docker Image Build
   ↓
Push Image to DockerHub
   ↓
Kubernetes Deployment Update
   ↓
Pods (2 replicas)
   ↓
Service (ClusterIP)
```

---

## 🧱 Architecture Overview

### Docker
Used to containerize the Django application.

### GitLab CI
Automates build and deployment.

### Kubernetes (kind)
Runs the application locally with:
- Namespace
- Deployment (2 replicas)
- Service (ClusterIP)

---

## 🧪 Kubernetes Setup

### Namespace
All application resources run inside a dedicated namespace:
```
notesapp
```
This keeps the application isolated from other workloads.

### Deployment
- Uses the Docker image pushed by the pipeline
- Runs 2 replicas for availability
- Kubernetes automatically recreates Pods if they fail
- Image is updated per commit (no latest tag)

**Key characteristics:**
- apps/v1 Deployment
- Label-based Pod selection
- Self-healing via ReplicaSet

### Service
- **Type:** ClusterIP
- Exposes the application inside the cluster
- Selects Pods using labels
- Used for:
  - Internal communication
  - Port-forwarding for local access

**Example access (local):**
```bash
kubectl port-forward svc/notes-app-service 8000:8000 -n notesapp
```

---

## 🚀 CI/CD Pipeline Overview

The pipeline runs in two stages:

### 1️⃣ Build Stage
- Pulls code from the repository
- Builds a Docker image
- Tags the image with the Git commit SHA
- Pushes the image to DockerHub

**Why this matters:**
- Every deployment is traceable
- Easy rollbacks
- No mutable latest tag

### 2️⃣ Deploy Stage
- Connects to the Kubernetes cluster using kubectl
- Creates namespace (idempotent)
- Updates Deployment image dynamically
- Applies Deployment and Service manifests
- Verifies rollout status

---

## 🔐 Pipeline Requirements

### GitLab Runner
The runner executing the pipeline must have:
- `docker`
- `kubectl`
- Network access to the Kubernetes API server

### GitLab CI Variables
The following variables must be configured in **GitLab → CI/CD → Variables**:

| Variable Name | Description |
|---------------|-------------|
| `DOCKERHUB_USERNAME` | DockerHub username |
| `DOCKERHUB_PASSWORD` | DockerHub password / token |
| `KUBECONFIG` | Base64-encoded kubeconfig |

⚠️ **Secrets are never stored in the repository**

---

## 🧠 Image Tagging Strategy

Images are tagged using:
```
<commit-short-sha>
```

**Example:**
```
myselfazeem/notes-app-k8s:a3f92bd
```

**Benefits:**
- Immutable deployments
- Easy rollback
- Clear audit trail

---

## 🛡️ Security Notes

- `kubectl port-forward` binds only to localhost
- The application is not exposed to the public internet
- Service type is ClusterIP (safe by default)
- No node ports or load balancers are used in local setup

---

## 🧩 Local Development Access

To access the app locally:

```bash
kubectl port-forward svc/notes-app-service 8000:8000 -n notesapp
```

Then open:
```
http://localhost:8000
```

Stopping the command immediately closes access.
