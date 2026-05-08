# AWS EKS Deployment Guide — CI/CD Pipeline with Kubernetes Monitoring

## 🚀 Project Overview

This guide explains how to deploy the CI/CD Kubernetes monitoring project to Amazon EKS using:

- Jenkins
- Docker
- Kubernetes
- Amazon EKS
- Amazon ECR
- Prometheus
- Grafana
- HPA (Horizontal Pod Autoscaler)

---

# 🏗️ Architecture

```text
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Prometheus + Grafana
```

---

# 📋 Prerequisites

Install the following tools:

| Tool | Purpose |
|---|---|
| AWS CLI | AWS authentication |
| kubectl | Kubernetes CLI |
| eksctl | EKS cluster creation |
| Docker Desktop | Build Docker images |

---

# 🔧 Step 1 — Install AWS CLI

Verify installation:

```bash
aws --version
```

---

# 🔧 Step 2 — Install eksctl

Verify installation:

```bash
eksctl version
```

---

# 🔧 Step 3 — Verify kubectl

```bash
kubectl version --client
```

---

# 🔐 Step 4 — Configure AWS CLI

Run:

```bash
aws configure
```

Provide:
- AWS Access Key
- AWS Secret Key
- Region → `ap-south-1`
- Output format → `json`

---

# ☸️ Step 5 — Create EKS Cluster

```bash
eksctl create cluster \
--name devops-cluster \
--region ap-south-1 \
--nodegroup-name devops-nodes \
--node-type t3.medium \
--nodes 2
```

---

# ✅ Step 6 — Verify EKS Cluster

```bash
kubectl get nodes
```

---

# 📦 Step 7 — Create Amazon ECR Repository

```bash
aws ecr create-repository --repository-name devops-app
```

---

# 🔐 Step 8 — Login to Amazon ECR

```bash
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

---

# 🐳 Step 9 — Build Docker Image

```bash
docker build -t devops-app:v1 .
```

---

# 🏷️ Step 10 — Tag Docker Image

```bash
docker tag devops-app:v1 <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:v1
```

---

# 🚀 Step 11 — Push Docker Image to ECR

```bash
docker push <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:v1
```

---

# ☸️ Step 12 — Update Kubernetes Deployment

Update `k8s-deployment.yaml`:

```yaml
image: <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:v1
```

Remove:

```yaml
imagePullPolicy: Never
```

---

# 🚀 Step 13 — Deploy Application to EKS

```bash
kubectl apply -f k8s-deployment.yaml
```

Verify:

```bash
kubectl get pods
```

---

# 🌐 Step 14 — Expose Application

Update Kubernetes service type:

```yaml
type: LoadBalancer
```

Apply configuration:

```bash
kubectl apply -f k8s-deployment.yaml
```

---

# 🌍 Step 15 — Get External URL

```bash
kubectl get svc
```

Open the external load balancer URL shown in EXTERNAL-IP.

---

# 📊 Step 16 — Deploy Prometheus

```bash
kubectl apply -f prometheus-deployment.yaml
```

---

# 📈 Step 17 — Deploy Grafana

```bash
kubectl apply -f grafana-deployment.yaml
```

---

# 🌐 Step 18 — Expose Grafana

Update Grafana service:

```yaml
type: LoadBalancer
```

Apply changes:

```bash
kubectl apply -f grafana-deployment.yaml
```

---

# 🔐 Step 19 — Login to Grafana

Default credentials:

```text
Username: admin
Password: admin
```

---

# 🔗 Step 20 — Configure Prometheus Data Source

Inside Grafana:

```text
Connections → Data Sources → Add Data Source
```

Choose:
- Prometheus

URL:

```text
http://prometheus-service:9090
```

---

# 📈 Step 21 — Verify Metrics

Prometheus query:

```text
app_requests_total
```

---

# ⚖️ Step 22 — Deploy Horizontal Pod Autoscaler

```bash
kubectl apply -f hpa.yaml
```

---

# 📊 Step 23 — Install Kubernetes Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify:

```bash
kubectl top nodes
```

---

# 🔥 Step 24 — Test Auto Scaling

```bash
kubectl run -i --tty load-generator --image=busybox -- sh
```

Inside pod:

```bash
while true; do wget -q -O- http://devops-service; done
```

Monitor scaling:

```bash
kubectl get hpa -w
```

---

# 🛠️ Useful Kubernetes Commands

## Get Pods

```bash
kubectl get pods
```

## Get Services

```bash
kubectl get svc
```

## Get Endpoints

```bash
kubectl get endpoints
```

## View Logs

```bash
kubectl logs deployment/devops-app
```

## Restart Deployment

```bash
kubectl rollout restart deployment devops-app
```

---

# 🚨 Common Troubleshooting

## Pod CrashLoopBackOff

```bash
kubectl logs <pod-name>
```

## Service Not Accessible

```bash
kubectl describe svc devops-service
kubectl get endpoints
```

## Prometheus Target DOWN

Verify metrics endpoint:

```text
http://localhost:8081/metrics
```

---

# 💰 Important AWS Cost Note

Delete cluster after testing:

```bash
eksctl delete cluster --name devops-cluster
```

---

# 🚀 Future Enhancements

Recommended next upgrades:

- Helm Charts
- Kubernetes Ingress
- GitHub Actions
- Loki Logging
- Terraform Infrastructure
- SSL/TLS
- ArgoCD GitOps

---

# 🎯 Resume Summary

Built and deployed a production-grade CI/CD pipeline integrating:

- Docker
- Kubernetes
- Amazon EKS
- Jenkins
- Prometheus
- Grafana
- HPA Auto Scaling
- Monitoring & Observability

---

# 🙌 Final Outcome

You now have a complete cloud-native DevOps portfolio project demonstrating:

✅ CI/CD  
✅ Kubernetes  
✅ Monitoring  
✅ Observability  
✅ Cloud Deployment  
✅ Auto Scaling  
✅ Troubleshooting Skills
