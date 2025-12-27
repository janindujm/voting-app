# 3-Tier Web Application Deployment on AWS EKS

This project demonstrates the deployment of a **3-tier web application** on **Amazon EKS (Elastic Kubernetes Service)** using **Terraform** for infrastructure provisioning and **Kubernetes** for container orchestration. The architecture includes a **React frontend**, **Node.js API backend**, and **MongoDB database** with persistent storage.

---

## Architecture Overview

- **Frontend:** React application deployed as a Kubernetes Deployment, exposed via a LoadBalancer service.
- **API:** Node.js/Express backend deployed as a Kubernetes Deployment, exposed via a LoadBalancer service.
- **Database:** MongoDB deployed as a StatefulSet with 3 replicas for high availability, using PersistentVolumeClaims (PVCs) for data persistence.

### Diagram

```
[Frontend LoadBalancer]
          |
      [Frontend Pods]
          |
      [API LoadBalancer]
          |
      [API Pods]
          |
     [MongoDB StatefulSet]
```

---

## Infrastructure Provisioning

Terraform is used to provision all required AWS resources:

1. **EC2 Client Machine:**  
   - Security group configured for SSH access.
   - Installed AWS CLI, kubectl, Docker, Java, and bash completion.
   - Terraform config: `client/ec2.tf`

2. **VPC & Networking:**  
   - Custom VPC with public and private subnets.
   - NAT gateway enabled.
   - Terraform config: `main.tf`

3. **EKS Cluster:**  
   - Deployed using `terraform-aws-modules/eks/aws`.
   - Cluster version: `1.29`
   - Managed node group with 2 t3.medium instances.
   - IRSA enabled for service accounts (e.g., EBS CSI driver).
   - Terraform config: `eks/`

4. **IAM Roles:**  
   - IAM roles for service accounts (IRSA) using module `iam-role-for-service-accounts-eks`.
   - Terraform config: `eks/iam.tf`

---

## Kubernetes Deployment

### MongoDB StatefulSet
- 3 replicas with PVCs for persistence.
- Anti-affinity rules to distribute pods across nodes.
- Configured as a replica set.

### API Deployment
- 2 replicas of Node.js API.
- Exposed via LoadBalancer.
- Environment variables configured for MongoDB connection and credentials.

### Frontend Deployment
- 2 replicas of React app.
- Exposed via LoadBalancer.
- Connects to API using environment variable for API URL.

### Secrets
- MongoDB credentials stored in `mongodb-secret`.

---

## Commands

### Terraform
```bash
terraform init
terraform plan
terraform apply
```

### Connect to EC2 Client
```bash
ssh -i <your-key.pem> ec2-user@<EC2_PUBLIC_IP>
```

### Update kubeconfig for EKS
```bash
aws eks --region us-east-1 update-kubeconfig --name my-eks-cluster
```

### Deploy Kubernetes Resources
```bash
kubectl apply -f manifests/
```

### Check Cluster Resources
```bash
kubectl get pods,svc,statefulsets,pvc -n cloudchamp
```

---

## Outputs

- **Frontend LoadBalancer URL:** `http://<frontend-elb-url>`
- **API LoadBalancer URL:** `http://<api-elb-url>`
- **EC2 SSH Command:** `ssh -i devops-project-1.pem ec2-user@<ec2-public-ip>`

---

## Tech Stack

- **Cloud:** AWS (EKS, VPC, EC2, ELB, IAM)
- **Infrastructure as Code:** Terraform
- **Container Orchestration:** Kubernetes
- **Frontend:** React
- **Backend:** Node.js / Express
- **Database:** MongoDB (StatefulSet)
- **CI/CD Ready:** Can integrate with Jenkins or GitHub Actions

---

## Notes

- This project demonstrates **high availability**, **load balancing**, and **persistent storage** in a cloud-native microservices setup.
- Designed for **learning and demonstration purposes**; for production, security rules (e.g., SSH access) should be restricted.

