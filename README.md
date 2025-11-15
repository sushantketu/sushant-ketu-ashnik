# sushant-ketu-ashnik Assignment
## Introduction:

Introduction & Goals

 Kubernetes Cluster Setup

 Nginx Ingress Basics & Helm

 Application Deployment

 Ansible Automation & Playbook

 TLS Implementation

 Troubleshooting & Learnings


## Configuration Details
A Github repo: sushant-ketu-ashnik
VS Code on the Window Local system 
 3 Linux Ubuntu 22.04 Nodes installed on Oracle virtual box.
 1 Master and 2 Worker Nodes
 Space: 
  3 GB RAM each node
  35 GB HDD for Master Node 
  30 GB HDD for Worker Nodes


## Project Overview

This repo documents a live Kubernetes setup on Oracle VirtualBox (Ubuntu 22.04) using 1 master & 2 worker nodes.  
All configuration, automation (Ansible), and resource manifests are included for reproducibility.

## High-Level Steps


1. Provision VMs (OS details, resources)
2. Install container runtime and dependencies
3. Bootstrap Kubernetes cluster (kubeadm)
4. Apply a CNI network plugin (Calico)
5. Deploy example workloads (nginx)
6. Troubleshoot and verify cluster health

Please see docs and code folders for deeper details and YAMLs.


## Git Frequent usable commands;

git status                     # To see what's changed

git add .                      # To stage all files for commit

git commit -m "Descriptive message for what I added/changed"

git push                       #To push on our GitHub repo



Step 1: Kubernetes Local 3-Node Cluster Setup
Created k8s manifests files under k8s-manifests folder;
named: nginx-deployment.yaml & nginx-service.yaml

Install on Master and Worker Nodes:
 
sudo apt-get update
sudo apt-get install -y containerd
sudo systemctl start containerd
sudo systemctl enable containerd
sudo apt-get install -y kubeadm kubelet kubectl
sudo systemctl enable kubelet

Initialize Master Node:
 
sudo kubeadm init --apiserver-advertise-address=<master-ip> --pod-network-cidr=192.168.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Join Worker Nodes:
 Run on each worker:

 
  sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

Step 2: Install Nginx Ingress Controller Using Helm
Add repo and update:

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update


Install ingress controller:
 
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace kube-system

Create basic ingress resource manifest to route traffic under / path.


Step 3: Deployed 'Hello World' Application
Sample Deployment manifest hello-deployment.yaml:
kubectl apply -f k8s-manifests/hello-deployment.yaml

Added a service to expose Hello World (k8s-manifests/hello-service.yaml)
 kubectl apply -f k8s-manifests/hello-service.yaml

 Created a new ingress file:
 k8s-manifests/hello-ingress.yaml
 kubectl apply -f k8s-manifests/hello-ingress.yaml

# Verify pods are running
kubectl get pods

# Verify services are created
kubectl get svc

# Verify ingress is created and routing
kubectl get ingress


Step 4: Ansible Playbook Structure
 
Running the Playbook
 
ansible-playbook -i inventory playbook.yml -c local

Inventory file may contain:
localhost ansible_connection=local

This command runs the playbook locally, authenticating to the Kubernetes cluster using the local kubeconfig.

Verifying Deployment

Check the TLS secret:
kubectl get secret myapp-tls -n default -o yaml

Check ingress and app status:
 kubectl get ingress,deploy,svc

Notes
 The playbook performs all operations locally without SSH to Kubernetes nodes.

 It is idempotent: TLS certs are only regenerated if missing.

 Ensure all paths in the playbook are correct relative to your run location on Windows.


Step 5: README.md Sample Content
 
# Kubernetes & Nginx Ingress Deployment Automation

## Prerequisites
- 3 Linux VMs for Kubernetes Master node and Workers node
- Nginx 1.28.0
- Kubernetes 1.22+
- Ansible 2.9+
- Helm 3

## Setup Steps
1. Prepare Kubernetes 3-node cluster using kubeadm.
2. Run Ansible playbook to install nginx ingress and deploy the Hello World app.
3. Access the app via ingress.

## Running Playbook
ansible-playbook -i inventory.ini deploy.yml


## Troubleshooting/Learnings
- Check node status: `kubectl get nodes`
- Check ingress controller pods: `kubectl get pods -n kube-system`
- Logs: `kubectl logs <pod> -n kube-system`

+I learnt a lot while creating/implementing the complete Kubernetes Cluster setup and Ansible automation via Playbook and everything via Helm commands.
 
-I faced some initial challenges with setting up the Kubernetes master node, which gave me valuable hands-on learning.

-Networking and pod communication needed some adjustments. I carefully checked the configurations and fixed the issues step by step.

+Ensuring the container runtime worked well with Kubernetes required checking versions and tuning settings.

-The cluster initialization had a few bumps due to system prerequisites like disabling swap and enabling certain kernel modules. Resolving these helped me understand system dependencies better.

-Running Kubernetes on VirtualBox with limited resources caused some performance issues that I overcame by optimizing resource allocation.

+I used kubectl logs and system logs to quickly find problems and apply fixes, improving my troubleshooting skills.

++Overall, these challenges helped me build a practical approach to setting up and managing Kubernetes clusters smoothly.


