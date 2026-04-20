# 🚀 AI Campus Assistant – Continuous Deployment (CD)

This repository contains Kubernetes manifests and GitOps configuration for deploying the **AI Campus Assistant** application on **Azure Kubernetes Service (AKS)** using **ArgoCD**.

The deployment follows a structured DevOps workflow integrating containerization, GitOps, and monitoring.

---

# 🧠 Architecture Overview

```text id="zrlx8t"
Developer → GitHub (Application Repository)
        → Jenkins (CI Pipeline)
        → Docker Image Build
        → Azure Container Registry (ACR)
        ↓
GitHub (CD Repository – this repo)
        ↓
ArgoCD (GitOps)
        ↓
Azure Kubernetes Service (AKS)
        ↓
NGINX Ingress Controller
        ↓
End User Access
```

---

# 📁 Repository Structure

```text id="q0djxb"
ai-campus-cd/
│
├── deployment.yaml     # Defines application deployment
├── service.yaml        # Exposes application within cluster
├── ingress.yaml        # Routes external traffic
└── README.md
```

---

# ⚙️ Prerequisites

Ensure the following components are configured:

* Azure Subscription
* Azure Kubernetes Service (AKS) Cluster
* Azure Container Registry (ACR)
* Jenkins CI pipeline (image build & push)
* ArgoCD installed on AKS
* Helm installed (for monitoring setup)

---

# 🔐 Step 1: Connect to AKS Cluster

```bash id="zt04xk"
az aks get-credentials \
--resource-group ai-campus-rg \
--name ai-campus-aks
```

Verify cluster access:

```bash id="t9smbw"
kubectl get nodes
```

---

# 🔑 Step 2: Create Kubernetes Secret

The application requires an API key to run.

```bash id="4afdrx"
kubectl create secret generic openai-secret \
--from-literal=OPENAI_API_KEY=<your-api-key>
```

Validate:

```bash id="0bs8t3"
kubectl get secrets
```

---

# 📦 Step 3: Deploy Application Resources

Apply Kubernetes manifests:

```bash id="66hwb8"
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Verify deployment:

```bash id="htybnx"
kubectl get pods
kubectl get svc
```

---

# 🌐 Step 4: Configure Service Exposure

The application is exposed using a Kubernetes Service:

* Type: `NodePort`
* Port Mapping:

```text id="1db6x3"
Port 80 → TargetPort 8000
```

Ensure the NodePort is accessible via Azure Network Security Group (NSG).

---

# 🔍 Step 5: Validate Application Locally

Use port-forwarding for validation:

```bash id="20z7gl"
kubectl port-forward pod/<pod-name> 8090:8000
```

Access in browser:

```text id="kkph3d"
http://localhost:8090
```

---

# 🌍 Step 6: Install NGINX Ingress Controller

Deploy ingress controller:

```bash id="u7d7s1"
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Verify:

```bash id="eeg70j"
kubectl get pods -n ingress-nginx
```

---

# 🌐 Step 7: Configure Ingress

Apply ingress configuration:

```bash id="w88zwi"
kubectl apply -f ingress.yaml
```

Verify:

```bash id="u0s2i4"
kubectl get ingress
```

Retrieve external IP:

```bash id="r8d1kq"
kubectl get svc -n ingress-nginx
```

Access application via external IP.

---

# 🔄 Step 8: Install and Configure ArgoCD

## Install ArgoCD

```bash id="s6u8k1"
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## Access ArgoCD UI

```bash id="c4qgho"
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```text id="5r30i5"
http://localhost:8080
```

---

## Retrieve Login Credentials

```bash id="mbmj9z"
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d
```

---

## Create Application in ArgoCD

Configure:

* Repository URL → this repository
* Path → `/`
* Cluster → default
* Namespace → default

---

# 🔄 Deployment Workflow

1. Code changes pushed to application repository
2. Jenkins builds and pushes Docker image to ACR
3. CD repository reflects updated configuration
4. ArgoCD detects changes and syncs automatically
5. Application is deployed to AKS

---

# 📊 Step 9: Monitoring Setup

Monitoring is implemented using Prometheus and Grafana.

## Install Monitoring Stack

```bash id="6n9n0b"
helm install monitoring prometheus-community/kube-prometheus-stack \
--namespace monitoring \
--create-namespace
```

---

## Access Grafana

```bash id="a0ub0l"
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Open:

```text id="v9k7al"
http://localhost:3000
```

---

# 🎯 Final Outcome

* Application deployed on AKS
* GitOps-based deployment using ArgoCD
* External access via NGINX Ingress
* Monitoring enabled with Prometheus & Grafana
* Scalable and production-ready architecture

---

# 📚 Additional Documentation

* CI Pipeline → Refer CI repository
* Troubleshooting → `docs/ci-errors.md`, `docs/cd-errors.md`

---

# 👨‍💻 Author

Ajay Pasunoori
