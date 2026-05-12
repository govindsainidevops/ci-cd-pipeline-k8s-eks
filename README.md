# 🚀 CI/CD Pipeline Deployment on AWS EKS

## 📌 Project Overview

This project demonstrates a complete DevOps workflow using:

* Docker
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
Docker Build
      ↓
Amazon ECR
      ↓
Amazon EKS
      ↓
Kubernetes Deployment
      ↓
Prometheus + Grafana
      ↓
HPA Auto Scaling
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

| Setting   | Reason                 |
| --------- | ---------------------- |
| t3.small  | Lower vCPU usage       |
| 1 Node    | Avoid quota exhaustion |
| us-east-1 | Stable region          |
| 2 AZs     | Required by eksctl     |

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

Expected:

```text
Login Succeeded
```

---

# 🐳 Step 5 — Build Docker Image

```bash
docker build -t devops-app:v1 .
```

---

# 🏷️ Step 6 — Tag Docker Image

```bash
docker tag devops-app:v1 336984083625.dkr.ecr.us-east-1.amazonaws.com/devops-app:v1
```

---

# 🚀 Step 7 — Push Docker Image

```bash
docker push 336984083625.dkr.ecr.us-east-1.amazonaws.com/devops-app:v1
```

---

# ☸️ Step 8 — Kubernetes Deployment File

Create:

```text
k8s-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: devops-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: devops-app

  template:
    metadata:
      labels:
        app: devops-app

    spec:
      containers:
      - name: devops-app

        image: 336984083625.dkr.ecr.us-east-1.amazonaws.com/devops-app:v1

        ports:
        - containerPort: 5000

        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"

          limits:
            cpu: "500m"
            memory: "512Mi"

---

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

# 🚀 Step 9 — Deploy Application

```bash
kubectl apply -f k8s-deployment.yaml
```

---

# 🔍 Step 10 — Verify Deployment

```bash
kubectl get pods
kubectl get svc
```

---

# 🌍 Step 11 — Access Application

Wait for external IP:

```bash
kubectl get svc
```

Open:

```text
http://<EXTERNAL-IP>
```

---

# 📊 Step 12 — Prometheus Deployment

Create:

```text
prometheus-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: prometheus

spec:
  replicas: 1

  selector:
    matchLabels:
      app: prometheus

  template:
    metadata:
      labels:
        app: prometheus

    spec:
      containers:
      - name: prometheus

        image: prom/prometheus

        ports:
        - containerPort: 9090

---

apiVersion: v1
kind: Service

metadata:
  name: prometheus-service

spec:
  selector:
    app: prometheus

  ports:
    - port: 9090
      targetPort: 9090

  type: LoadBalancer
```

Apply:

```bash
kubectl apply -f prometheus-deployment.yaml
```

---

# 📈 Step 13 — Grafana Deployment

Create:

```text
grafana-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: grafana

spec:
  replicas: 1

  selector:
    matchLabels:
      app: grafana

  template:
    metadata:
      labels:
        app: grafana

    spec:
      containers:
      - name: grafana

        image: grafana/grafana

        ports:
        - containerPort: 3000

---

apiVersion: v1
kind: Service

metadata:
  name: grafana-service

spec:
  selector:
    app: grafana

  ports:
    - port: 3000
      targetPort: 3000

  type: LoadBalancer
```

Apply:

```bash
kubectl apply -f grafana-deployment.yaml
```

---

# 🔐 Step 14 — Login to Grafana

Default credentials:

```text
Username: admin
Password: admin
```

---

# 🔗 Step 15 — Configure Prometheus Data Source

Grafana → Connections → Data Sources → Add Data Source

Choose:

* Prometheus

URL:

```text
http://prometheus-service:9090
```

---

# 📈 Step 16 — Verify Metrics

Example query:

```text
up
```

---

# ⚖️ Step 17 — Configure Horizontal Pod Autoscaler (HPA)

Create:

```text
hpa.yaml
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: devops-hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: devops-app

  minReplicas: 1
  maxReplicas: 5

  metrics:
  - type: Resource

    resource:
      name: cpu

      target:
        type: Utilization
        averageUtilization: 50
```

---

# 🚀 Apply HPA

```bash
kubectl apply -f hpa.yaml
```

---

# 🔍 Verify HPA

```bash
kubectl get hpa
```

---

# 📊 Verify Metrics Server

```bash
kubectl get deployment metrics-server -n kube-system
```

---

# 🚨 Install Metrics Server (If Missing)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

# 📈 Verify Metrics

```bash
kubectl top nodes
kubectl top pods
```

---

# 🔥 Generate Load for HPA Testing

```bash
kubectl run -i --tty load-generator --image=busybox -- sh
```

Inside container:

```bash
while true; do wget -q -O- http://devops-service; done
```

---

# 📊 Watch Auto Scaling

```bash
kubectl get hpa -w
```

Monitor pods:

```bash
kubectl get pods -w
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

## Update kubeconfig

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name devops-cluster
```

---

## Verify Current Context

```bash
kubectl config get-contexts
```

---

## Verify ECR Images

```bash
aws ecr describe-images \
--repository-name devops-app \
--region us-east-1
```

---

# 💰 Cleanup

Delete cluster:

```bash
eksctl delete cluster \
--name devops-cluster \
--region us-east-1
```

---

# 🚀 Future Improvements

* Jenkins Automation
* Terraform
* ArgoCD GitOps
* Helm Charts
* Loki Logging
* AlertManager
* SSL/TLS Ingress
* Blue-Green Deployment

---

# 🎯 Resume-Worthy Skills Demonstrated

✅ Docker
✅ Kubernetes
✅ AWS EKS
✅ Amazon ECR
✅ Prometheus
✅ Grafana
✅ HPA
✅ Monitoring & Observability
✅ Cloud Deployment
✅ Kubernetes Auto Scaling
✅ Production Troubleshooting

---

# 🙌 Final Outcome

Successfully deployed a cloud-native application on AWS EKS with:

* Containerized deployment
* Monitoring stack
* Auto scaling
* Cloud-native architecture
* Kubernetes orchestration
* Production-style deployment workflow
