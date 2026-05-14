# Full Kubernetes (K8s) Installation on a Laptop

> Production-like Kubernetes setup using kubeadm + containerd on Ubuntu.

---

# Overview

This guide installs the **full Kubernetes cluster** on a laptop.

This is **NOT**:

- Minikube
    
- K3s
    
- Kind
    

You will run a real Kubernetes control plane similar to production environments.

---

# Recommended Specs

## OS

- Ubuntu 22.04 or 24.04
    

## Hardware

- RAM: 8GB minimum (16GB recommended)
    
- CPU: 4 cores+
    
- Storage: 40GB+
    

---

# Components Installed

- kubeadm
    
- kubelet
    
- kubectl
    
- containerd
    

---

# Architecture

```text
+---------------------------------------------------+
|                 Kubernetes Cluster                |
+---------------------------------------------------+
|                                                   |
|  Control Plane                                    |
|  ├── kube-apiserver                               |
|  ├── etcd                                         |
|  ├── scheduler                                    |
|  ├── controller-manager                           |
|                                                   |
|  Worker Components                                |
|  ├── kubelet                                      |
|  ├── kube-proxy                                   |
|  ├── containerd                                   |
|                                                   |
+---------------------------------------------------+
```

---

# 1. Disable Swap

Kubernetes requires swap to be disabled.

## Temporary

```bash
sudo swapoff -a
```

## Permanent

Edit fstab:

```bash
sudo nano /etc/fstab
```

Comment swap line:

```bash
#/swapfile none swap sw 0 0
```

---

# 2. Install containerd

Update packages:

```bash
sudo apt update
```

Install containerd:

```bash
sudo apt install -y containerd
```

Create config directory:

```bash
sudo mkdir -p /etc/containerd
```

Generate default config:

```bash
containerd config default | sudo tee /etc/containerd/config.toml
```

Restart service:

```bash
sudo systemctl restart containerd
```

Enable on boot:

```bash
sudo systemctl enable containerd
```

Check status:

```bash
sudo systemctl status containerd
```

---

# 3. Add Kubernetes Repository

Install dependencies:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Add signing key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | \
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

---

# 4. Install Kubernetes Packages

Update repositories:

```bash
sudo apt update
```

Install packages:

```bash
sudo apt install -y kubelet kubeadm kubectl
```

Prevent automatic upgrades:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Check versions:

```bash
kubectl version --client
kubeadm version
```

---

# 5. Initialize Kubernetes Cluster

Initialize cluster:

```bash
sudo kubeadm init
```

This process may take several minutes.

---

# 6. Configure kubectl

Create kube directory:

```bash
mkdir -p $HOME/.kube
```

Copy admin config:

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Fix permissions:

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Test access:

```bash
kubectl get nodes
```

---

# 7. Install Pod Networking

Without networking, Pods cannot communicate.

## Install Calico

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/master/manifests/calico.yaml
```

Check Pods:

```bash
kubectl get pods -A
```

---

# 8. Allow Scheduling on Control Plane

Since this is a single-node cluster:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

---

# 9. Verify Cluster

```bash
kubectl get nodes
```

Expected:

```text
NAME       STATUS   ROLES           AGE   VERSION
ubuntu     Ready    control-plane   5m    v1.30.x
```

---

# Deploy First Application

## Create nginx deployment

```bash
kubectl create deployment nginx --image=nginx
```

## Expose deployment

```bash
kubectl expose deployment nginx --type=NodePort --port=80
```

---

# Check Resources

```bash
kubectl get all
```

---

# Useful kubectl Commands

## View nodes

```bash
kubectl get nodes
```

## View pods

```bash
kubectl get pods
```

## View all namespaces

```bash
kubectl get ns
```

## Describe pod

```bash
kubectl describe pod <pod-name>
```

## View logs

```bash
kubectl logs <pod-name>
```

## Delete deployment

```bash
kubectl delete deployment nginx
```

---

# Install Kubernetes Dashboard (Optional)

Official documentation:

- [https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
    

---

# Cluster Components

## Control Plane

|Component|Purpose|
|---|---|
|kube-apiserver|Main API|
|etcd|Cluster database|
|scheduler|Assigns workloads|
|controller-manager|Maintains cluster state|

---

## Node Components

|Component|Purpose|
|---|---|
|kubelet|Manages Pods|
|kube-proxy|Networking|
|containerd|Runs containers|

---

# Difference from Minikube

|Feature|Full K8s|Minikube|
|---|---|---|
|Production-like|Yes|Limited|
|Real control plane|Yes|Partial|
|Multi-node support|Yes|Limited|
|Complexity|High|Low|
|Resource usage|High|Medium|

---

# Recommended Learning Path

1. Pods
    
2. Deployments
    
3. Services
    
4. ConfigMaps
    
5. Secrets
    
6. Volumes
    
7. Ingress
    
8. Helm
    
9. RBAC
    
10. Monitoring
    

---

# Troubleshooting

## Pods stuck in Pending

Check networking:

```bash
kubectl get pods -A
```

---

## Node NotReady

Restart kubelet:

```bash
sudo systemctl restart kubelet
```

---

## Check cluster info

```bash
kubectl cluster-info
```

---

# Optional Add-ons

- Helm
    
- MetalLB
    
- Prometheus
    
- Grafana
    
- ArgoCD
    
- Istio
    

---

# Cleanup

Reset cluster:

```bash
sudo kubeadm reset -f
```

Remove Kubernetes packages:

```bash
sudo apt remove -y kubeadm kubelet kubectl
```

---

# Official Documentation

- [https://kubernetes.io/](https://kubernetes.io/)
    
- [https://kubernetes.io/docs/](https://kubernetes.io/docs/)
    
- [https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)