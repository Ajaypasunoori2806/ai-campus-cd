# 🚀 AI Campus Assistant – Continuous Deployment (CD)

This repository contains Kubernetes manifests and GitOps configuration for deploying the **AI Campus Assistant** application on **Azure Kubernetes Service (AKS)** using **ArgoCD**.

The deployment follows a complete DevOps workflow integrating CI, containerization, GitOps, and monitoring.

---

# 🧠 Architecture Overview

```
Developer → GitHub (App Repo)
        → Jenkins (CI Pipeline)
        → Docker Image Build
        → Azure Container Registry (ACR)
        ↓
GitHub (CD Repo – this repo)
        ↓
ArgoCD (GitOps)
        ↓
Azure Kubernetes Service (AKS)
        ↓
Ingress Controller (NGINX)
        ↓
End User Access
```

---

# 📁 Repository Structure

```
ai-campus-cd/
│
├── deployment.yaml     # Application Deployment
├── service.yaml        # Service Exposure
├── ingress.yaml        # External Routing via NGINX
└── README.md
```

---

# ⚙️ Prerequisites

Ensure the following are ready:

* Azure Subscription
* AKS Cluster
* Azure Container Registry (ACR)
* Jenkins CI pipeline pushing images
* ArgoCD installed on cluster

---

# 📦 Deployment Components

## 🔹 1. Deployment

* Deploys container from ACR:

```
aicampusacr2806.azurecr.io/ai-campus-app:latest
```

* Exposes port `8000`
* Uses Kubernetes Secret for API key

---

## 🔹 2. Service

* Type: `NodePort` (used due to Azure Public IP limits)
* Maps:

```
Port 80 → TargetPort 8000
```

---

## 🔹 3. Ingress

* Uses NGINX Ingress Controller
* Routes external traffic to service
* Avoids multiple LoadBalancers (cost optimization)

---

# 🚀 ArgoCD Setup

## Step 1: Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## Step 2: Access ArgoCD

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```
http://localhost:8080
```

---

## Step 3: Get Login Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d
```

---

## Step 4: Create Application

* Repo URL → this repository
* Path → `/`
* Cluster → default
* Namespace → default

---

# 🔄 Deployment Flow

1. Developer pushes code
2. Jenkins builds Docker image
3. Image pushed to ACR
4. CD repo updated
5. ArgoCD detects change
6. Application deployed automatically

---

# 📊 Monitoring Setup

Monitoring implemented using:

* Prometheus
* Grafana

## Install Monitoring Stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
--namespace monitoring \
--create-namespace
```

---

## Access Grafana

```bash
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Open:

```
http://localhost:3000
```

---

# ⚠️ Key Challenges & Solutions

* Azure Public IP limit issue
* Kubernetes Secret missing
* Pod crashes due to dependency issues
* ArgoCD sync troubleshooting
* Port conflicts during testing

👉 Detailed issues documented in:

```
docs/ci-errors.md
docs/cd-errors.md
```

---

# 🎯 Final Outcome

✔ Application deployed on AKS
✔ GitOps-based CD pipeline
✔ Automated deployments using ArgoCD
✔ Monitoring enabled with Prometheus & Grafana
✔ Cost optimized using Ingress

---

# 👨‍💻 Author

Ajay Pasunoori
