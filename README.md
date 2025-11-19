🚀 HelloApp – GitOps Deployment with ArgoCD & EKS

This repository contains a minimal example of deploying a HelloApp application to Amazon EKS using a GitOps workflow powered by ArgoCD.

ArgoCD continuously monitors this repository and automatically syncs any changes to the Kubernetes cluster.

📦 Repository Structure:

- deployment.yml — Kubernetes Deployment manifest for the application.

- service.yml (optional) — Kubernetes Service (LoadBalancer) for external access.

- README.md — Project documentation.

⚙️ Technologies Used:

- Amazon EKS (Elastic Kubernetes Service)

- ArgoCD

- Kubernetes

- GitHub

- CI/CD + GitOps

🔁 GitOps Workflow:

- You push updates to the YAML files in this repository.

- ArgoCD detects changes automatically.

- ArgoCD syncs the application state to the EKS cluster.

- The application updates instantly with no manual kubectl apply needed.

🚀 Deployment Steps (Summary):

Install ArgoCD on the EKS cluster.

Add the repository to ArgoCD:

argocd repo add https://github.com/<your-repo>.git


Create an Application in ArgoCD:

Path: .

Cluster URL: https://kubernetes.default.svc

Namespace: default

Click Sync to deploy the application.

🌐 Accessing the Application:

Once synced, retrieve the LoadBalancer URL:

kubectl get svc

Open the EXTERNAL-IP in your browser.
