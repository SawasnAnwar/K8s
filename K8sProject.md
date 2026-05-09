
## 1) Install k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

## 2) Check service

```bash
sudo systemctl status k3s --no-pager
```

## 3) Check nodes

```bash
sudo k3s kubectl get nodes
```
## 4) Get node token

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```
# K3s Cluster Setup Notes

## Check K3s API Port

```bash
sudo ss -lntp | grep 6443
```

Output:

```text
LISTEN 0 4096 *:6443 *:* users:(("k3s-server",pid=5593,fd=12))
```

This confirms that the K3s server is running and listening on port `6443`.

---

## Check Cluster Information

```bash
sudo k3s kubectl cluster-info
```

Output:

```text
Kubernetes control plane is running at https://127.0.0.1:6443

CoreDNS is running at
https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

Metrics-server is running at
https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy
```

This confirms the Kubernetes cluster is active.

---

## Incorrect Worker Join Command

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.81.132:6443 K3S_TOKEN=<TOKEN> sh -
```

Error:

```text
-bash: TOKEN: No such file or directory
```

Reason:
- The token was written inside `< >`
- In Linux, `< >` are interpreted as redirection operators

---

## Correct Worker Join Command

```bash
curl -sfL https://get.k3s.io |K3S_URL=https://192.168.81.132:6443 K3S_TOKEN=<TOKEN> sh -
```

Important:
- Do NOT use `< >` around the token
- Run this command on the Worker Node, not on the K3s server itself

---

## Check Nodes

```bash
sudo k3s kubectl get nodes
```

# Add Worker Node to K3s Cluster

Exactly — you need 2 separate Linux VMs.

- VM1 = Control Plane (already done)
- VM2 = Worker Node (new VM)

---

# Create Second VM (Worker)

In:
- VirtualBox
- VMware
- Hyper-V

Create another Ubuntu VM or clone the current one.

Give it a different hostname, for example:

```text
worker1
```

---

# Configure Network

Set VM2 to use the same network mode as VM1:
- Bridged Adapter
- or same NAT Network

Start VM2 and check its IP address:

```bash
hostname -I
```

---

# Change Hostname (If Cloned)

```bash
sudo hostnamectl set-hostname worker1
```

Then:
- logout/login
- or reboot

---

# Test Connectivity from VM2 to VM1

Ping the control-plane node:

```bash
ping -c 3 192.168.81.132
```

Test Kubernetes API port:

```bash
nc -zv 192.168.81.132 6443
```

---

# Get Token from VM1 (Control Plane)

Run on VM1:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

---

# Join Worker Node

Run this command on VM2 only:

```bash
curl -sfL https://get.k3s.io | \
K3S_URL=https://192.168.81.132:6443 \
K3S_TOKEN='<TOKEN_FROM_VM1>' \
sh -
```

Important:
- Replace `<TOKEN_FROM_VM1>` with the real token
- Do NOT include `< >` around the actual token

---

# Verify Cluster Nodes

Run on VM1:

```bash
sudo k3s kubectl get nodes -o wide
```

Expected output:

```text
sawsan-virtual-machine   control-plane
worker1                  worker
```

Run this on your VM to fully remove k3s:

```bash
sudo /usr/local/bin/k3s-uninstall.sh
sudo /usr/local/bin/k3s-agent-uninstall.sh 2>/dev/null || true
```

Then verify it is gone:

```bash
systemctl status k3s --no-pager
systemctl status k3s-agent --no-pager
kubectl config get-contexts
```

If scripts are missing, use:

```bash
sudo systemctl stop k3s k3s-agent 2>/dev/null || true
sudo systemctl disable k3s k3s-agent 2>/dev/null || true
sudo rm -f /etc/systemd/system/k3s*.service
sudo rm -f /etc/systemd/system/k3s*.service.env
sudo systemctl daemon-reload
sudo rm -rf /etc/rancher/k3s /var/lib/rancher/k3s
sudo rm -f /usr/local/bin/k3s /usr/local/bin/kubectl /usr/local/bin/crictl /usr/local/bin/ctr
```
# Start Existing Minikube Profile as Multi-Node

## Stop Old Cluster

```bash
minikube stop
```

---

## Delete Old Single-Node Cluster

```bash
minikube delete
```

---

## Start Minikube with 3 Nodes

```bash
minikube start -p minikube --nodes 3 --driver=docker
```

---

## Verify Nodes

```bash
kubectl config use-context minikube
kubectl get nodes -o wide
```
# Minikube Multi-Node Cluster Status

## Check Cluster Nodes

```bash
kubectl get nodes -o wide
```

Current output:

```text
NAME           STATUS     ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION      CONTAINER-RUNTIME
minikube       Ready      control-plane   3m19s   v1.35.1   192.168.49.2   <none>        Debian GNU/Linux 12 (bookworm)   6.8.0-107-generic   docker://29.2.1
minikube-m02   NotReady   <none>          2m17s   v1.35.1   192.168.49.3   <none>        Debian GNU/Linux 12 (bookworm)   6.8.0-107-generic   docker://29.2.1
minikube-m03   NotReady   <none>          76s     v1.35.1   192.168.49.4   <none>        Debian GNU/Linux 12 (bookworm)   6.8.0-107-generic   docker://29.2.1
```

---

# Explanation

- `minikube`
  - control-plane node
  - status = `Ready`

- `minikube-m02`
  - worker node
  - status = `NotReady`

- `minikube-m03`
  - worker node
  - status = `NotReady`

This is normal immediately after cluster startup.

Worker nodes may need:
- more startup time
- image pulls
- kubelet initialization

---

# Wait and Recheck

```bash
kubectl get nodes -w
```

or:

```bash
watch kubectl get nodes
```

Expected result after a few minutes:

```text
minikube       Ready
minikube-m02   Ready
minikube-m03   Ready
```

---

# If Nodes Stay NotReady

Check node details:

```bash
kubectl describe node minikube-m02
```

```bash
kubectl describe node minikube-m03
```

Check pods:

```bash
kubectl get pods -A
```
