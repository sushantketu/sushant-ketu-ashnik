# sushant-ketu-ashnik
OS Details;
3 Linux Ubuntu 22.04 Nodes installed on Oracle virtual box.
1 Master and 2 Worker Nodes

# sushant-ketu-ashnik

## Project Overview

This repo documents a live-looking Kubernetes setup on Oracle VirtualBox (Ubuntu 22.04) using 1 master & 2 worker nodes.  
All configuration, automation (Ansible), and resource manifests are included for reproducibility.

## High-Level Steps

1. Provision VMs (OS details, resources)
2. Install container runtime and dependencies
3. Bootstrap Kubernetes cluster (kubeadm)
4. Apply a CNI network plugin (Calico/Flannel)
5. Deploy example workloads (nginx)
6. Troubleshoot and verify cluster health

Please see docs and code folders for deeper details and YAMLs.


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

Step 3: Deploy 'Hello World' Application
Sample Deployment manifest hello-world-deployment.yaml:

text
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: containous/whoami
        ports:
        - containerPort: 80
Service manifest hello-world-service.yaml:

text
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  selector:
    app: hello-world
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
Ingress manifest example:

text
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 80

Step 4: Ansible Playbook Structure
text
- hosts: master
  become: yes
  tasks:
    - name: Install Helm
      shell: |
        curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 |  
      args:
        creates: /usr/local/bin/helm

    - name: Add ingress-nginx repo
      shell: helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

    - name: Update Helm repos
      shell: helm repo update

    - name: Install nginx ingress controller
      shell: |
        helm upgrade --install nginx-ingress ingress-nginx/ingress-nginx --namespace kube-system

    - name: Deploy Hello World app
      kubernetes.core.k8s:
        state: present
        definition: '{{ lookup("file", "hello-world-deployment.yaml") }}'

    - name: Deploy Hello World service
      kubernetes.core.k8s:
        state: present
        definition: '{{ lookup("file", "hello-world-service.yaml") }}'

    - name: Deploy Ingress
      kubernetes.core.k8s:
        state: present
        definition: '{{ lookup("file", "hello-world-ingress.yaml") }}'

- hosts: master
  become: yes
  tasks:
    - name: Generate self-signed TLS cert
      shell: |
        openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout tls.key -out tls.crt \
        -subj "/CN=example.com/O=example.com"
      args:
        creates: tls.crt

    - name: Create TLS secret
      kubernetes.core.k8s:
        state: present
        namespace: default
        definition: |
          apiVersion: v1
          kind: Secret
          metadata:
            name: tls-secret
          data:
            tls.crt: {{ lookup('file', 'tls.crt') | b64encode }}
            tls.key: {{ lookup('file', 'tls.key') | b64encode }}
          type: kubernetes.io/tls

    - name: Deploy Ingress with TLS
      kubernetes.core.k8s:
        state: present
        definition: |
          apiVersion: networking.k8s.io/v1
          kind: Ingress
          metadata:
            name: hello-world-ingress-tls
            annotations:
              nginx.ingress.kubernetes.io/rewrite-target: /
          spec:
            tls:
            - hosts:
              - example.com
              secretName: tls-secret
            rules:
            - host: example.com
              http:
                paths:
                - path: /
                  pathType: Prefix
                  backend:
                    service:
                      name: hello-world
                      port:
                        number: 80

Step 5: README.md Sample Content
text
# Kubernetes & Nginx Ingress Deployment Automation

## Prerequisites
- 3 Linux VMs for Kubernetes Master and Workers
- Kubernetes 1.22+
- Ansible 2.9+
- Helm 3

## Setup Steps
1. Prepare Kubernetes 3-node cluster using kubeadm.
2. Run Ansible playbook to install nginx ingress and deploy the Hello World app.
3. Access the app via ingress (http://example.com).

## Running Playbook
ansible-playbook -i inventory.ini deploy.yml

text

## Troubleshooting
- Check node status: `kubectl get nodes`
- Check ingress controller pods: `kubectl get pods -n kube-system`
- Logs: `kubectl logs <pod> -n kube-system`

## Notes
- TLS uses a self-signed certificate generated via playbook.

Step 6: Interview Explanation Highlights
Explain kubeadm initialization flow & pod networking

Show Helm-based ingress-nginx installation for simplicity

Discuss Ansible modular playbook and idempotency

Describe TLS termination at ingress with self-signed cert

Highlight troubleshooting and documentation practices

Step 7: Presentation Outline for Interview
Introduction & Goals

Kubernetes Cluster Setup

Nginx Ingress Basics & Helm

Application Deployment

Ansible Automation & Playbook

TLS Implementation

Demo & Access

Troubleshooting & Learnings