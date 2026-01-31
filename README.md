# Teaf_DevOps_Assessment

# Teaf_DevOps_Assessment

Simple Java App – CI/CD on Kubernetes with Jenkins + Kaniko

This repository contains a simple Spring Boot application and a complete CI/CD pipeline using **Jenkins**, **Kubernetes**, and **Kaniko** to automatically build, push, and deploy the application.

---

## Application Overview

- Endpoint: `GET /`  
  Returns: `hello [name]`
- Environment Variable: `YOUR_NAME`
- Health Endpoint: `GET /actuator/health`

---

## Prerequisites (Tools Setup)

Before you begin, ensure you have a Linux environment (or WSL) and sufficient privileges (sudo) for installing packages.

### 1. Update Packages
```bash
sudo apt update
```

### 2. Install Java 17 (JRE Headless)
Why JRE Headless?
- No GUI (server-friendly)
- Lightweight
- Ideal for servers and Docker containers

```bash
sudo apt install -y openjdk-17-jre-headless
java -version
```

### 3. Install Maven
```bash
sudo apt install -y maven
mvn -v
```

### 4. Install Docker
```bash
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
docker --version
```

Add your user to the docker group so you can run Docker without sudo:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 5. Install kubectl (Client Only)
```bash
sudo apt install -y kubectl
kubectl version --client
```
Note: This displays only the kubectl client version (no cluster connection).

### 6. Install Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 7. Install kind
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

---

## Create Kubernetes Cluster (kind)

Create the cluster:
```bash
kind create cluster --name teafdevops
kubectl get nodes
```

Install NGINX Ingress Controller (for kind):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=180s
```

---

## Docker Hub Login (Manual Test)
```bash
docker login -u <DOCKERHUB_USERNAME>
# Use a Docker Hub Access Token as the password.
```

---

## Repository Structure
```
.
├── Dockerfile
├── Jenkinsfile
├── README.md
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── simple-java-app/
    ├── pom.xml
    └── src/
```

---

## Kubernetes Manifests

- `deployment.yaml` – defines pods, replicas, and container image
- `service.yaml` – exposes the app internally (ClusterIP)
- `ingress.yaml` – exposes the app externally using NGINX Ingress

Deploy manually (optional):
```bash
kubectl apply -f k8s/
```

---

## Build & Push Docker Image (Manual Test)
```bash
docker build -t <DOCKERHUB_USERNAME>/simple-java-app:1.0.0 .
docker push <DOCKERHUB_USERNAME>/simple-java-app:1.0.0
```

---

## Install Jenkins on Kubernetes (Helm)

Add chart repo and install:
```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
kubectl create namespace jenkins
helm install jenkins jenkins/jenkins -n jenkins
```

Retrieve Jenkins Admin Password:
```bash
kubectl exec --namespace jenkins -it svc/jenkins -c jenkins -- \
  /bin/cat /run/secrets/additional/chart-admin-password && echo
```

Port-forward to access Jenkins locally:
```bash
kubectl -n jenkins port-forward svc/jenkins 8081:8080
```
Open in browser:
- http://127.0.0.1:8081

---

## CI/CD Pipeline (Jenkinsfile)

Pipeline Stages
- Build & Test (Maven)
- Build & Push Docker Image (Kaniko)
- Deploy to Kubernetes

Why Kaniko?
- Kaniko builds container images without requiring a Docker daemon, making it ideal for Kubernetes-based Jenkins agents.

---

## Required Jenkins Credentials & Secrets

Jenkins Credential (Docker Hub)
- Kind: Username with password
- ID: `dockerhub-creds`
- Username: Docker Hub username
- Password: Docker Hub Access Token

Kubernetes Secret for Kaniko:
```bash
kubectl create secret docker-registry dockerhub-config \
  --namespace default \
  --docker-username=<DOCKERHUB_USERNAME> \
  --docker-password=<DOCKERHUB_TOKEN> \
  --docker-server=https://index.docker.io/v1/
```

---

## Troubleshooting

- Jenkinsfile Not Found  
  Use Pipeline script from SCM  
  Script Path: `Jenkinsfile`

- RBAC Forbidden (403)  
  If Jenkins needs cluster-admin:
  ```bash
  kubectl create clusterrolebinding jenkins-cluster-admin \
    --clusterrole=cluster-admin \
    --serviceaccount=jenkins:jenkins
  ```

- Agent Launch Error (MalformedURLException)  
  Set Jenkins URL in Jenkins Location and configure Kubernetes Cloud:
  - Jenkins URL = external Jenkins URL
  - Jenkins Tunnel = `jenkins-agent.jenkins:50000`

- ImagePullBackOff  
  ```bash
  docker pull <DOCKERHUB_USERNAME>/simple-java-app:latest
  kubectl describe deploy simple-java-app | grep -i image
  ```

---

## Verification

Check resources:
```bash
kubectl get deploy
kubectl get pods
kubectl get svc
```

Port-forward the service to test the app locally:
```bash
kubectl port-forward svc/simple-java-app-svc 8080:80
```

Open in browser:
- http://127.0.0.1:8080/
- http://127.0.0.1:8080/actuator/health
