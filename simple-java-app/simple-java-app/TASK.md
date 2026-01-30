# DevOps Trainee Task: Deploying the Simple Java App

Welcome! This document outlines your task to build a complete CI/CD pipeline and deploy the `simple-java-app` on Kubernetes.

## Your Mission

Your goal is to automate the building, testing, and deployment of this application using modern DevOps tools and practices.

## Core Tasks

### 1. Jenkins and Kubernetes Setup

- **Install Jenkins:** Set up a Jenkins controller. You can run it on a VM, a Docker container, or even locally for this task.
- **Set up a Kubernetes Cluster:** Create a Kubernetes cluster. You can use Minikube, Kind or whatever you prefer.
- **Configure Jenkins with Kubernetes:**
    - Install the Kubernetes plugin in Jenkins.
    - Configure the plugin to connect to your Kubernetes cluster.
    - Set up Jenkins to dynamically provision agents (workers) on your Kubernetes cluster. This will allow your pipeline jobs to run in a scalable and isolated environment.

### 2. Prepare the Application for Kubernetes

- **Create Kubernetes Manifests:** In this repository, create a `k8s` directory and add the following Kubernetes manifest files:
    - `deployment.yaml`: To define how to deploy the `simple-java-app`.
    - `service.yaml`: To expose the application within the cluster.
    - `ingress.yaml`: To expose the application outside the cluster. You will need to install an Ingress controller (like NGINX Ingress Controller) in your cluster for this to work.

### 3. Build a CI/CD Pipeline

- **Create a `Jenkinsfile`:** In the root of this repository, create a `Jenkinsfile` that defines the CI/CD pipeline. The pipeline should have the following stages:
    - **Build:** Check out the code, compile it, and run unit tests using Maven.
    - **Build Docker Image:** Build the Docker image for the application.
    - **Push Docker Image:** Push the Docker image to a container registry ( Docker Hub).
    - **Deploy to Kubernetes:** Apply the Kubernetes manifests to deploy the application to your cluster.

## Bonus Task (GitOps with ArgoCD)

For a more advanced setup, implement a GitOps workflow:

- **Set up a GitOps Repository:** Create a new Git repository that will store the Kubernetes manifests for the application.
- **Install ArgoCD:** Install ArgoCD in your Kubernetes cluster.
- **Configure ArgoCD:** Configure ArgoCD to monitor your GitOps repository and automatically sync the application's manifests to the cluster.
- **Update the CI/CD Pipeline:** Modify your Jenkins pipeline to update the Kubernetes manifests in the GitOps repository instead of directly applying them to the cluster.

Good luck!
