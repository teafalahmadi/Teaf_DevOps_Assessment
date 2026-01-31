# Teaf_DevOps_Assessment

###############################################################################
# Simple Java App – CI/CD on Kubernetes with Jenkins + Kaniko
###############################################################################

# This repository contains a simple Spring Boot application and a complete
# CI/CD pipeline using Jenkins, Kubernetes, and Kaniko to build, push,
# and deploy the application automatically.

###############################################################################
# Application Overview
###############################################################################

# GET /                -> returns: hello [name]
# Environment Variable -> YOUR_NAME
# Health Endpoint      -> GET /actuator/health

###############################################################################
# Prerequisites (Tools Setup)
###############################################################################

# 1) Update packages
sudo apt update


# 2) Install Java 17 (JRE Headless)
sudo apt install -y openjdk-17-jre-headless
java -version

# Reason for choosing JRE Headless:
# - No GUI (server-friendly)
# - Lightweight
# - Ideal for servers and Docker containers


# 3) Install Maven
sudo apt install -y maven
mvn -v


# 4) Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
docker --version

# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker


# 5) Install kubectl (client only)
sudo apt install -y kubectl
kubectl version --client
# Shows kubectl client version only and does not connect to any cluster


# 6) Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version


# 7) Install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version

###############################################################################
# Create Kubernetes Cluster (kind)
###############################################################################

# Create cluster
kind create cluster --name teafdevops
kubectl get nodes


# Install NGINX Ingress Controller (kind)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml


# Wait for ingress controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=180s

###############################################################################
# Docker Hub Login (Manual Test)
###############################################################################

docker login -u <DOCKERHUB_USERNAME>
# Use Docker Hub Access Token as password

###############################################################################
# Repository Structure
###############################################################################

# .
# ├── Dockerfile
# ├── Jenkinsfile
# ├── README.md
# ├── k8s/
# │   ├── deployment.yaml
# │   ├── service.yaml
# │   └── ingress.yaml
# └── simple-java-app/
#     ├── pom.xml
#     └── src/...

###############################################################################
# Kubernetes Manifests
###############################################################################

# deployment.yaml -> defines pods, replicas, and container image
# service.yaml    -> exposes the app internally (ClusterIP)
# ingress.yaml    -> exposes the app externally via NGINX Ingress

# Deploy application manually (optional)
kubectl apply -f k8s/

###############################################################################
# Build & Push Docker Image (Manual Test)
###############################################################################

docker build -t <DOCKERHUB_USERNAME>/simple-java-app:1.0.0 .
docker push <DOCKERHUB_USERNAME>/simple-java-app:1.0.0

###############################################################################
# Install Jenkins on Kubernetes (Helm)
###############################################################################

# Add Jenkins Helm repository
helm repo add jenkins https://charts.jenkins.io
helm repo update

# Install Jenkins
kubectl create namespace jenkins
helm install jenkins jenkins/jenkins -n jenkins

###############################################################################
# Retrieve Jenkins Admin Password
###############################################################################

kubectl exec --namespace jenkins -it svc/jenkins -c jenkins -- \
  /bin/cat /run/secrets/additional/chart-admin-password && echo

###############################################################################
# Access Jenkins (Port Forward)
###############################################################################

kubectl -n jenkins port-forward svc/jenkins 8081:8080
# Open in browser:
# http://127.0.0.1:8081

###############################################################################
# Jenkins Integration with Kubernetes (Kubernetes Agents)
###############################################################################

# Jenkins dynamically creates Kubernetes Pods as agents for each pipeline run.
# Benefits:
# - Isolation (each build in its own Pod)
# - Better resource utilization
# - Scalability
# - Lightweight Jenkins controller

###############################################################################
# CI/CD Pipeline (Jenkinsfile)
###############################################################################

# Pipeline stages:
# 1) Build & Test (Maven)
# 2) Build & Push Docker Image (Kaniko)
# 3) Deploy to Kubernetes

# Kaniko is used to build Docker images without requiring a Docker daemon.

###############################################################################
# Required Jenkins Credentials / Secrets
###############################################################################

# Jenkins Credential (Docker Hub):
# Kind     : Username with password
# ID       : dockerhub-creds
# Username : Docker Hub username
# Password : Docker Hub Access Token

# Kubernetes Secret for Kaniko
kubectl create secret docker-registry dockerhub-config \
  --namespace default \
  --docker-username=<DOCKERHUB_USERNAME> \
  --docker-password=<DOCKERHUB_TOKEN> \
  --docker-server=https://index.docker.io/v1/

###############################################################################
# Troubleshooting (Triple-Shooting)
###############################################################################

# 1) Jenkinsfile not found
# Fix:
# - Use Pipeline script from SCM
# - Script Path: Jenkinsfile

# 2) No Kubernetes cloud was found
# Fix:
# - Install Kubernetes plugin
# - Configure Kubernetes cloud in Jenkins

# 3) Forbidden (403) when testing Kubernetes connection
kubectl create clusterrolebinding jenkins-cluster-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=jenkins:jenkins

# 4) Failed to launch agent (MalformedURLException)
# Fix:
# - Set Jenkins URL in Jenkins Location
# - Configure Kubernetes Cloud:
#   Jenkins URL   -> external Jenkins URL
#   Jenkins Tunnel -> jenkins-agent.jenkins:50000

# 5) ImagePullBackOff
docker pull <DOCKERHUB_USERNAME>/simple-java-app:latest
kubectl describe deploy simple-java-app | grep -i image

# 6) Port-forward issues
kubectl -n jenkins get pods
kubectl get pods
kubectl -n jenkins port-forward svc/jenkins 8081:8080
kubectl port-forward svc/simple-java-app-svc 8080:80

# 7) Ingress not working
kubectl -n ingress-nginx get pods

###############################################################################
# Verification
###############################################################################

kubectl get deploy
kubectl get pods
kubectl get svc

# Access application
kubectl port-forward svc/simple-java-app-svc 8080:80
# http://127.0.0.1:8080/
# http://127.0.0.1:8080/actuator/health
###############################################################################