# 🚀 Todo App – EKS GitOps Deployment *(ArgoCD + Terraform + CircleCI)*

A complete production-ready GitOps pipeline for deploying a React Todo application on Amazon EKS, provisioned using Terraform, continuously delivered via ArgoCD, and automated through CircleCI.
This repository documents the full lifecycle: containerization, infrastructure provisioning, Kubernetes deployment, GitOps automation, and CI/CD integration.

## 📌 Table of Contents

1. 🏗️ Architecture Overview
2. 🐳 Local Docker Testing
3. 🌐 Infrastructure Deployment (Terraform)
4. ☸️ Kubernetes Deployment
5. 🎯 ArgoCD Setup (GitOps Control Plane)
6. 🔄 CircleCI – CI/CD Automation
7. 📸 Screenshots
8. 👤 Author

## Architecture Overview

This project implements a realistic, cloud-native DevOps architecture:
- ⚛️ React Frontend
- 🐳 Docker Multi-Stage Image
- ☸️ EKS (AWS Managed Kubernetes)
- 🟪 Terraform IaC
- 🎯 ArgoCD GitOps Deployment
- 🔄 CircleCI CI/CD Pipeline
- 📦 DockerHub Registry
- 🧩 GitHub (App + Manifests repos)

---
## 🐳 Local Docker Testing

### 🔧 Multi-Stage Dockerfile
- Stage 1: Build React app using Node
- Stage 2: Serve production build via Nginx
- Commands
    docker build -t sanketdesai09/todo-react-app:latest .
    docker run -d -p 3000:80 sanketdesai09/todo-react-app:latest

### ⚠️ Issues Found & Resolved

| Issue                         | Cause                        | Fix                             |
|-------------------------------|------------------------------|----------------------------------|
| package.json missing          | Missing dependency install   | Ran `npm install`                |
| package.json & lock mismatch  | Corrupted dependency tree    | Deleted & regenerated fresh      |


---
## 🌐 Infrastructure Deployment (Terraform)

### 🏗️ Provisioned components:
- VPC (public + private subnets)
- Internet Gateway & NAT
- EKS Cluster (v1.28)
- Managed Node Group
- IAM Roles, SGs
- Remote backend (S3 + DynamoDB)

### 🛠️ Commands
- terraform init
- terraform validate
- terraform plan
- terraform apply --auto-approve

### 🔍 Verify Cluster
- aws eks --region us-east-1 update-kubeconfig --name todo-eks-cluster
- kubectl get nodes

---
## ☸️ Kubernetes Deployment

### 📦 Applied manifests:
- kubectl apply -f k8s-manifests/deployment.yaml
- kubectl apply -f k8s-manifests/service.yaml
- kubectl apply -f k8s-manifests/hpa.yaml

### 🔍 Verification commands:
- kubectl get pods
- kubectl get svc

---
## 🎯 ArgoCD Setup (GitOps Control Plane)

### ⚙️ Install
- kubectl create namespace argocd
- kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

### 🌍 Expose ArgoCD (Internet-Facing LoadBalancer)
- kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
-	kubectl get svc -n argocd 

### ⚠️ *Initial issue: ArgoCD created an internal LB due to private subnets.*
- Fix: Add annotation to enforce public LB.

kubectl patch svc argocd-server -n argocd -p '{
  "metadata": {
    "annotations": {
      "service.beta.kubernetes.io/aws-load-balancer-scheme": "internet-facing"
    }
  },
  "spec": {
    "type": "LoadBalancer"
  }
}'

### 🔐 Retrieve Admin Password
- kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

### 🧩 ArgoCD Application Config (manual)
- Connected manifests repo
- Enabled:
  Auto-Sync
  Self-Healing
  Prune Resources

---
## 🔄 CircleCI – CI/CD Automation

### 🛠️ Pipeline workflow:

- Build Docker image
- Push to DockerHub
- Clone manifests repo
- Update image tag using sed
- Commit & push to GitHub
- ArgoCD auto-sync deploys latest build

### 🔑 Required CircleCI Environment Variables
| Variable | Purpose |
| -------- | ------- |
| DOCKERHUB_USERNAME |	DockerHub login |
| DOCKERHUB_PASSWORD	| DockerHub token |
| GITHUB_TOKEN	| Push updates to manifests repo |
| MANIFEST_REPO |	GitOps repo URL |
| config.yaml | Highlights |

---
## 📸 Screenshots

### 1. Docker Build & Local Testing
- Docker build output

<img width="1002" height="109" alt="Screenshot 2025-11-14 141930" src="https://github.com/user-attachments/assets/dfcab501-9a99-4cf0-ba02-b01d334fbab4" />

- Running container locally

<img width="1435" height="112" alt="Screenshot 2025-11-14 142059" src="https://github.com/user-attachments/assets/c7a07ff9-ab9c-48d1-aa21-2aaa4335e2c6" />

- Application in browser

<img width="1914" height="332" alt="image" src="https://github.com/user-attachments/assets/f50c74df-83e4-4231-801e-6524d0fb6a52" />

---
### 2. Terraform Infrastructure

- Terraform plan 

<img width="940" height="340" alt="image" src="https://github.com/user-attachments/assets/577d0a61-1703-4ff8-b3f3-00bc6d3407fc" />

- Terraform apply

<img width="940" height="254" alt="image" src="https://github.com/user-attachments/assets/b4eca8e8-2f88-4a1f-9e1b-45e6ebd60f96" />

---
### 3. Kubernetes Deployment

- Pods running

<img width="1331" height="83" alt="image" src="https://github.com/user-attachments/assets/ce7fdf58-01a6-4b7d-bef9-e28b54e3bcf4" />

- Services

<img width="1341" height="86" alt="image" src="https://github.com/user-attachments/assets/fbb30da2-f6ba-44ba-95b0-be05c583ef07" />

---
### 4. ArgoCD Setup

- ArgoCD pods

<img width="915" height="202" alt="Screenshot 2025-11-13 112412" src="https://github.com/user-attachments/assets/d86162d3-487a-44a0-84e7-c2a5b0c1855b" />

- LoadBalancer details

<img width="1628" height="265" alt="Screenshot 2025-11-13 132340" src="https://github.com/user-attachments/assets/ab0a6bb6-ea81-4a44-bf06-02271a4d606b" />

- ArgoCD App Syncing

<img width="1669" height="556" alt="Screenshot 2025-11-13 132415" src="https://github.com/user-attachments/assets/2e06502c-faaf-4b32-b2e4-98bd110f910e" />

---
### 5. CircleCI Pipeline

- Entire workflow success

<img width="1895" height="910" alt="Screenshot 2025-11-13 132238" src="https://github.com/user-attachments/assets/38c3440f-c9ce-4b94-825d-0bacdbc07d08" />

- Image pushed to DockerHub

<img width="1222" height="672" alt="Screenshot 2025-11-13 132328" src="https://github.com/user-attachments/assets/32b31b15-669b-4398-9adb-83333b0df689" />

---

## 👤 Author
- Sanket Desai – *DevOps & AWS Engineer*
- GitOps | Kubernetes | Terraform | AWS | CI/CD | Cloud Infrastructure
