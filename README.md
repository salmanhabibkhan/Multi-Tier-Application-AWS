# Multi-Tier Application Deployment on Kubernetes

This README.md provides installation and deployment instructions for the multi-tier application on Kubernetes, including setup for a Helm chart, configuring TLS with a self-signed certificate, and details on Security Contexts, Horizontal Pod Autoscaling (HPA), and Role-Based Access Control (RBAC). It also covers assumptions and considerations made during the deployment.

# Prerequisites

1 - Kubernetes Cluster: Ensure you have a Kubernetes cluster set up and kubectl configured on Local (Minikube) or AWS (EKS). 
2 - Helm: Helm should be installed and configured.
3 - Cert-Manager (optional): For automatic TLS certificate management if not using self-signed certificates.

# Helm chart Installation and Clone Code

git clone https://github.com/salmanhabibkhan/Multi-Tier-Application-AWS.git

cd Multi-Tier-Application-AWS

Start the Cluster on Local or AWS.

# Local - Minikube start

Run this cmd after Minikube start - minikube addons enable ingress

# EKS

eksctl create cluster --name three-tier-cluster --region us-east-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes

# Deploy the Helm Chart: Run the following command to install the chart:

helm install multi-tier-app ./k8s-manifests

# Verify the Installation: Check that all pods, services, and other resources have been created:

kubectl get pods,svc,deployments

# Setting Up TLS for Ingress (Self-Signed Certificate)

Generate a Self-Signed Certificate and Key:
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=your-domain.com"

Create a Kubernetes Secret for TLS:
kubectl create secret tls frontend-tls --cert=tls.crt --key=tls.key

# Update the Ingress Configuration: In your values.yaml file, add the following to configure Ingress with TLS:

ingress:
  enabled: true
  tls:
    - hosts:
        - your-domain.com
      secretName: frontend-tls

# Apply Ingress Changes: If you modified the Helm chart values, run:

helm upgrade multi-tier-app ./k8s-manifests

# Configuration Details

Security Contexts

Security contexts are defined to enforce least privilege and ensure secure execution of containers. The following settings are used:

    Run as Non-Root User: Both backend and MySQL components run as a non-root user (UID 1000), which reduces the risk of privilege escalation.
    Read-Only Root Filesystem: Configured for backend to restrict writable permissions to specific directories.

Here’s a sample README.md file for your project that includes the required information.
Multi-Tier Application Deployment on Kubernetes

This document provides installation and deployment instructions for the multi-tier application on Kubernetes, including setup for a Helm chart, configuring TLS with a self-signed certificate, and details on Security Contexts, Horizontal Pod Autoscaling (HPA), and Role-Based Access Control (RBAC). It also covers assumptions and considerations made during the deployment.
Prerequisites

    Kubernetes Cluster: Ensure you have a Kubernetes cluster set up and kubectl configured.
    Helm: Helm should be installed and configured.
    Cert-Manager (optional): For automatic TLS certificate management if not using self-signed certificates.

1. Helm Chart Installation
Step-by-Step Deployment

    Clone the Repository:

    bash

git clone https://github.com/your-repo/multi-tier-application.git
cd multi-tier-application

Configure Values: Modify values.yaml as necessary to set database credentials, replica counts, and other parameters:

yaml

backend:
  env:
    DB_HOST: mysql
    DB_USER: root
    DB_PASSWORD: my-secret-pw
    DB_NAME: mydb
mysql:
  rootPassword: my-secret-pw
  dbName: mydb

Deploy the Helm Chart: Run the following command to install the chart:

bash

helm install multi-tier-app ./helm

Verify the Installation: Check that all pods, services, and other resources have been created:

bash

    kubectl get pods,svc,deployments

2. Setting Up TLS for Ingress (Self-Signed Certificate)

To secure the ingress for the frontend, follow these steps to create a self-signed certificate:

    Generate a Self-Signed Certificate and Key:

    bash

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=your-domain.com"

Create a Kubernetes Secret for TLS:

bash

kubectl create secret tls frontend-tls --cert=tls.crt --key=tls.key

Update the Ingress Configuration: In your values.yaml file, add the following to configure Ingress with TLS:

yaml

ingress:
  enabled: true
  tls:
    - hosts:
        - your-domain.com
      secretName: frontend-tls

Apply Ingress Changes: If you modified the Helm chart values, run:

bash

    helm upgrade multi-tier-app ./helm

# Configuration Details
Security Contexts

Security contexts are defined to enforce least privilege and ensure secure execution of containers. The following settings are used:

    Run as Non-Root User: Both backend and MySQL components run as a non-root user (UID 1000), which reduces the risk of privilege escalation.
    Read-Only Root Filesystem: Configured for backend to restrict writable permissions to specific directories.

Example from values.yaml:

yaml

backend:
  securityContext:
    runAsUser: 1000
    readOnlyRootFilesystem: true
mysql:
  securityContext:
    runAsUser: 1000

# Horizontal Pod Autoscaler (HPA)

The backend service has an HPA to scale automatically based on CPU usage. This helps the application handle varying load efficiently.

    Min Replicas: 2
    Max Replicas: 10
    Target CPU Utilization: 75%

# RBAC (Role-Based Access Control)

RBAC is used to restrict permissions. We created a service account for the backend and assigned a role that allows it access to specific resources like ConfigMaps and Secrets.

Assumptions and Considerations

    Persistent Storage: We use Persistent Volume Claims (PVC) for MySQL to ensure data is retained even if the database pod restarts or redeploys.
    Ingress with Self-Signed TLS: Self-signed certificates are used for demonstration; in production, consider using Cert-Manager for automated certificate management with Let’s Encrypt.
    Non-Root User Execution: All containers run as non-root users to follow best practices for container security.
    Scalability: HPA is configured only for the backend, as the frontend (React + NGINX) generally requires fewer resources and can be scaled manually if necessary.