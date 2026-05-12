# 🚀 CI/CD Pipeline Deployment on AWS EKS

## 📌 Project Overview

This project demonstrates a complete DevOps workflow using:

* Docker
* Jenkins
* Kubernetes
* AWS EKS
* Amazon ECR
* Prometheus
* Grafana
* Horizontal Pod Autoscaler (HPA)

The application is containerized, pushed to Amazon ECR, and deployed on AWS EKS with monitoring and scaling enabled.

---

# 🏗️ Architecture

```text
Developer Push
      ↓
GitHub Repository
      ↓
Jenkins Pipeline
      ↓
Docker Build
      ↓
Amazon ECR
      ↓
Amazon EKS
      ↓
Prometheus + Grafana Monitoring
```

---

# 📋 Prerequisites

Install the following tools:

| Tool           | Purpose              |
| -------------- | -------------------- |
| AWS CLI        | AWS Authentication   |
| kubectl        | Kubernetes CLI       |
| eksctl         | EKS Cluster Creation |
| Docker Desktop | Build Docker Images  |
| Jenkins        | CI/CD Pipeline       |

---

# 🔐 Step 1 — Configure AWS CLI

```bash
aws configure
```

Provide:

* AWS Access Key
* AWS Secret Access Key
* Region → `us-east-1`
* Output format → `json`

Verify:

```bash
aws sts get-caller-identity
```

---

# ☸️ Step 2 — Create EKS Cluster

## ✅ Recommended Working Configuration

```bash
eksctl create cluster \
--name devops-cluster \
--region us-east-1 \
--zones us-east-1a,us-east-1b \
--nodegroup-name devops-nodes \
--node-type t3.small \
--nodes 1
```

---

# 📌 Why This Configuration?

| Setting   | Reason                            |
| --------- | --------------------------------- |
| t3.small  | Lower vCPU usage                  |
| 1 Node    | Avoid quota exhaustion            |
| us-east-1 | Stable region                     |
| 2 AZs     | Required by newer eksctl versions |

---

# ✅ Verify Cluster Creation

```bash
kubectl get nodes
```

Expected Output:

```text
STATUS = Ready
```

---

# 📦 Step 3 — Create Amazon ECR Repository

```bash
aws ecr create-repository \
--repository-name devops-app \
--region us-east-1
```

---

# 🔐 Step 4 — Login to Amazon ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 336984083625.dkr.ecr.us-east-1.amazonaws.com
```

Expected Output:

```text
Login Succeeded
```

---

# 🐳 Step 5 — Build Docker Image

Inside project directory:

```bash
docker build -t devops-app:v1 .
```

---

# 🏷️ Step 6 — Tag Docker Image

```bash
docker tag devops-app:v1 336984083625.dkr.ecr.us-east-1.amazonaws.com/devops-app:v1
```

---

# 🚀 Step 7 — Push Docker Image to ECR

```bash
docker push 336984083625.dkr.ecr.us-east-1.amazonaws.com/devops-app:v1
```

---

# ☸️ Step 8 — Update Kubernetes Deployment

Update image inside `k8s-deployment.yaml`

```yaml
image: 336984083625.dkr.ecr.us-east-1.amazonaws.com/devops-app:v1
```

Remove:

```yaml
imagePullPolicy: Never
```

---

# 🌐 Step 9 — Kubernetes Service Configuration

Use `LoadBalancer` service type.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: devops-service

spec:
  selector:
    app: devops-app

  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000

  type: LoadBalancer
```

---

# 🚀 Step 10 — Deploy Application to EKS

```bash
kubectl apply -f k8s-deployment.yaml
```

---

# 🔍 Step 11 — Verify Deployment

```bash
kubectl get pods
kubectl get svc
```

---

# 🌍 Step 12 — Access Application

Wait for external IP:

```bash
kubectl get svc
```

Open:

```text
http://<EXTERNAL-IP>
```

---

# 📊 Step 13 — Deploy Prometheus

```bash
kubectl apply -f prometheus-deployment.yaml
```

---

# 📈 Step 14 — Deploy Grafana

```bash
kubectl apply -f grafana-deployment.yaml
```

---

# 🔐 Step 15 — Login to Grafana

Default Credentials:

```text
Username: admin
Password: admin
```

---

# 🔗 Step 16 — Configure Prometheus Data Source

Grafana → Connections → Data Sources → Add Data Source

Choose:

* Prometheus

URL:

```text
http://prometheus-service:9090
```

---

# 📈 Step 17 — Verify Metrics

Example Query:

```text
app_requests_total
```

---

# ⚖️ Step 18 — Deploy HPA

```bash
kubectl apply -f hpa.yaml
```

---

# 📊 Step 19 — Verify Metrics Server

```bash
kubectl top nodes
```

---

# 🔥 Step 20 — Test Auto Scaling

```bash
kubectl run -i --tty load-generator --image=busybox -- sh
```

Inside container:

```bash
while true; do wget -q -O- http://devops-service; done
```

Monitor:

```bash
kubectl get hpa -w
```

---

# 🛠️ Useful Commands

## Get Pods

```bash
kubectl get pods
```

## Get Services

```bash
kubectl get svc
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

# 🚨 Troubleshooting

## No Nodes Found

Update kubeconfig:

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name devops-cluster
```

---

## Check Current Context

```bash
kubectl config get-contexts
```

---

## Verify AWS Quotas

```bash
aws service-quotas get-service-quota \
--service-code ec2 \
--quota-code L-1216C47A
```

---

# 💰 Cleanup

Delete cluster after practice:

```bash
eksctl delete cluster \
--name devops-cluster \
--region us-east-1
```

---

# 🚀 Future Improvements

* Jenkins Full Automation
* Terraform
* ArgoCD GitOps
* Helm Charts
* Loki Logging
* AlertManager
* SSL/TLS
* Kubernetes Ingress

---

# 🎯 Resume-Worthy Skills Demonstrated

✅ Docker
✅ Kubernetes
✅ AWS EKS
✅ Amazon ECR
✅ Jenkins
✅ Prometheus
✅ Grafana
✅ HPA
✅ Cloud Troubleshooting
✅ Monitoring & Observability
✅ CI/CD Pipeline

---

# 🙌 Final Outcome

Successfully deployed a cloud-native DevOps application on AWS EKS with:

* CI/CD concepts
* Container orchestration
* Monitoring
* Scaling
* Cloud deployment
* Troubleshooting experience
