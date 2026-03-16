# AWS 3‑Tier Architecture Deployment using Terraform, Docker, and Kubernetes

## Overview

This guide explains how to deploy a 3‑tier architecture on AWS using Terraform, Docker, and Kubernetes. It includes domain configuration and TLS certificates using ACME.

Architecture:

* Presentation Layer (Frontend)
* Application Layer (Backend APIs)
* Data Layer (Database)

Tools Used:

* AWS
* Terraform
* Docker
* Kubernetes (EKS)
* GitHub
* ACM / Let's Encrypt (ACME)

---

# Table of Contents

1. Prerequisites
2. AWS IAM Setup
3. Terraform Installation
4. Project Repository Structure
5. Infrastructure Provisioning with Terraform
6. Docker Container Build and Push
7. Kubernetes Cluster Setup (EKS)
8. Kubernetes Deployment
9. Domain Configuration
10. SSL Certificate using ACME
11. Ingress Configuration
12. Validation
13. Security Best Practices

---

# 1. Prerequisites

Ensure the following are available:

* AWS Account
* IAM User with programmatic access
* Terraform installed
* Docker installed
* kubectl installed
* AWS CLI installed
* Git installed

Verify installations:

```
terraform -v
aws --version
docker --version
kubectl version --client
```

---

# 2. AWS IAM Setup

Create IAM user for Terraform.

Steps:

1. Login to AWS Console
2. Navigate to IAM
3. Create User
4. Enable Programmatic Access
5. Attach Policy

Required policies:

* AdministratorAccess (for lab)

Save:

* Access Key
* Secret Key

Configure locally:

```
aws configure
```

Enter:

```
Access Key
Secret Key
Region
Output format json
```

---

# 3. Terraform Installation

Linux:

```
sudo apt install terraform
```

Verify:

```
terraform -v
```

---

# 4. Repository Structure

Example structure:

```
project
│
├── terraform
│   ├── vpc.tf
│   ├── eks.tf
│   ├── security.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── docker
│   ├── frontend
│   └── backend
│
├── kubernetes
│   ├── frontend-deployment.yaml
│   ├── backend-deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
└── README.md
```

---

# 5. Infrastructure Provisioning with Terraform

Initialize terraform

```
cd terraform
terraform init
```

Plan

```
terraform plan
```

Apply

```
terraform apply
```

This will create:

* VPC
* Subnets
* Internet Gateway
* Security Groups
* EKS Cluster
* Node Groups

---

# 6. Docker Container Build

Build frontend

```
docker build -t frontend:1.0 .
```

Build backend

```
docker build -t backend:1.0 .
```

Tag images

```
docker tag frontend:1.0 <aws_account>.dkr.ecr.<region>.amazonaws.com/frontend:1.0
```

Push to ECR

```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin <aws_account>.dkr.ecr.region.amazonaws.com


docker push <repo>
```

---

# 7. Kubernetes Cluster Access

Update kubeconfig

```
aws eks update-kubeconfig --region region --name cluster-name
```

Verify

```
kubectl get nodes
```

---

# 8. Kubernetes Deployment

Deploy backend

```
kubectl apply -f backend-deployment.yaml
```

Deploy frontend

```
kubectl apply -f frontend-deployment.yaml
```

Create services

```
kubectl apply -f service.yaml
```

---

# 9. Domain Configuration

Purchase domain

Configure Route53

Create Hosted Zone

Add record

Example:

```
app.example.com
```

Point to LoadBalancer DNS

---

# 10. SSL Certificates with ACME

Install cert-manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

Create ClusterIssuer

Example:

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
 name: letsencrypt-prod
spec:
 acme:
   email: admin@example.com
   server: https://acme-v02.api.letsencrypt.org/directory
   privateKeySecretRef:
     name: letsencrypt-prod
   solvers:
   - http01:
       ingress:
         class: nginx
```

---

# 11. Ingress Configuration

Example ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: app-ingress
 annotations:
   cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
 tls:
 - hosts:
   - app.example.com
   secretName: tls-secret
 rules:
 - host: app.example.com
   http:
     paths:
     - path: /
       pathType: Prefix
       backend:
         service:
           name: frontend
           port:
             number: 80
```

Apply

```
kubectl apply -f ingress.yaml
```

---

# 12. Validation

Check pods

```
kubectl get pods
```

Check services

```
kubectl get svc
```

Check ingress

```
kubectl get ingress
```

Open browser

```
https://app.example.com
```

---

# 13. Security Best Practices

* Use IAM roles for service accounts
* Use private subnets for worker nodes
* Enable AWS WAF
* Enable encryption at rest
* Use secrets manager

---

# Conclusion

You have successfully deployed a production‑style 3‑tier architecture using Terraform, Docker, Kubernetes, and ACME certificates on AWS.
