# 3-Tier Kubernetes Project

## Overview
This project demonstrates the deployment of a 3-tier architecture (frontend, backend, and database) on Kubernetes using Terraform for infrastructure as code (IaC) and AWS as the cloud provider.

---

## Prerequisites
Make sure the following tools are installed and configured on your system:

- **AWS account**: Set up AWS credentials with proper access.
- **PEM file**: Required for SSH access to the EC2 instance.
- **Docker**: To build and manage container images.
- **Terraform**: For managing infrastructure as code.
- **kubectl**: To manage Kubernetes clusters.
- **eksctl**: For setting up Amazon EKS clusters.

---

## Setup Instructions

```bash
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf install terraform
+> 1. Install Terraform 

+> 2. Configure AWS Credentials
source .env
ssh -i <pem_filename> ubuntu@<ipaddress>
git clone https://github.com/Rajatsaxenabhu/3_tier_kubernetes_project.git

+> 3 Install Docker 
    sudo apt-get update
    sudo apt install docker.io
    docker ps
    sudo chown $USER /var/run/docker.sock
    cd frontend
    docker build -t frontend-image .
    cd ../backend
    docker build -t backend-image .

+> 4. Install kubectl:
    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin
    kubectl version --short --client

+> 5. Install eksctl:
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version

+> 6. Setup EKS Cluster:
    eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
    aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
    kubectl get nodes

+> 7. Run Manifests:
    kubectl create namespace workshop
    kubectl apply -f .
    kubectl delete -f .

+> 8. Install AWS Load Balancer:
    curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
    
    aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
    
    eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster 
    --approve
    
    eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2

+> 9.  Deploy AWS Load Balancer Controller
    
    sudo snap install helm --classic
    helm repo add eks https://aws.github.io/eks-charts
    helm repo update eks
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=3-tier-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
    kubectl get deployment -n kube-system aws-load-balancer-controller
    kubectl apply -f full_stack_lb.yaml
