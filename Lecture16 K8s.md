# Question

## How can a standalone Pod be managed by a ReplicaSet?

Yes — if you have a standalone Pod and want a ReplicaSet to “adopt” it, the Pod must match the ReplicaSet’s selector labels.

### Example

#### Existing Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    app: nginx

spec:
  containers:
    - name: nginx
      image: nginx
```

#### ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: myrs

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx
```

Because the Pod label:

```yaml
app: nginx
```

matches the ReplicaSet selector:

```yaml
matchLabels:
  app: nginx
```

the ReplicaSet can adopt and manage that Pod.

## Important Notes

- The Pod must not already be controlled by another controller.
- Labels must exactly match the ReplicaSet selector.
- The Pod template inside the ReplicaSet should also match.

## Conclusion

“It can be connected if it has the same label.”

More accurately:

> A ReplicaSet can adopt a standalone Pod if the Pod labels match the ReplicaSet selector.

# Flask Kubernetes   
make docker file and image and push image to Docker hub and Kubernetes it 


```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, Kubernetes!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

# Docker file Example name it Dockerfile

```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY . .

RUN pip install flask

EXPOSE 5000

CMD ["python", "app.py"]
```
# Build Docker Image

```bash

docker build -t sawsana/flask-app-test .

```
# Check Docker Images

```bash
docker images
```

Example output:

```text
IMAGE                           ID             DISK USAGE   CONTENT SIZE   EXTRA
python:3.9-slim                 085da638e1b8        122MB             0B
sawsanA/flask-app-test:latest   63800204eeb7        233MB             0B
```
# Login to Docker Hub

```bash
docker login -u sawsana
```

After entering the password:

```text
Login Succeeded
```


# Push Docker Image to Docker Hub


```bash
docker push sawsana/flask-app-test:latest
```

Output:

```text
The push refers to repository [docker.io/sawsana/flask-app-test]

fee76718bb96: Pushed
5456d38998d9: Pushed
9465b6fec5fb: Pushed

latest: digest: sha256:d6668f772532dde3fa149427e0677126ed17c7f1dcdc45f2865f184f20a3f68a
size: 1788
```

## Result

The Docker image was successfully pushed to Docker Hub.

# Create Kubernetes Deployment YAML

```bash
kubectl create deployment flask-app \
  --image=sawsana/flask-app-test:latest \
  --replicas=2 \
  -o yaml \
  --dry-run=client > deployment.yaml
```
# deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: flask-app
  labels:
    app: flask-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: flask-app

  template:
    metadata:
      labels:
        app: flask-app

    spec:
      containers:
        - name: flask-app-test
          image: sawsana/flask-app-test:latest
          resources: {}
```
# Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

Output:

```text
deployment.apps/flask-app created
```
# Check Kubernetes Deployment

```bash
kubectl get deployments
```

Or:

```bash
kubectl get deployments.apps
```

Output:

```text
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
flask-app   2/2     2            2           97s
```

## Result

- Deployment name: `flask-app`
- Replicas running: `2/2`
- Application is successfully deployed
# Check Running Pods

```bash
kubectl get pods
```

Output:

```text
NAME                         READY   STATUS    RESTARTS   AGE
flask-app-7b95b5974c-64fcs   1/1     Running   0          2m50s
flask-app-7b95b5974c-stptk   1/1     Running   0          2m50s
```

## Result

- Two Pods are running successfully.
- Status is `Running`.
- Each Pod is ready (`1/1`).

## how access this app from out side cluster ????

# Generate NodePort Service YAML

```bash
kubectl expose deployment flask-app \
  --type=NodePort \
  --port=5000 \
  --target-port=5000 \
  -o yaml \
  --dry-run=client > service.yaml
```


```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: flask-app
  name: flask-app
spec:
  ports:
  - port: 5000
    protocol: TCP
    targetPort: 5000
    nodePort: 30007
  selector:
    app: flask-app
  type: NodePort
status:
  loadBalancer: {}
```
# Apply Service

```bash
kubectl apply -f service.yaml
```

Output:

```text
service/flask-app created
```
# Check Services

```bash
kubectl get svc
```

Output:

```text
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
flask-app    NodePort    10.110.135.216   <none>        5000:30007/TCP   71s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          11d
```
# Check Kubernetes Nodes Details

```bash
kubectl get nodes -o wide
```

Output:

```text
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   11d   v1.34.3   172.30.1.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-110-generic   containerd://2.2.1
node01         Ready    <none>          11d   v1.34.3   172.30.2.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-110-generic   containerd://2.2.1
```
# Test the Application

```bash
curl 172.30.1.2:30007
```

Output:

```text
Hello, Kubernetes!
```
#  Deployment Strategy

![[66ca3b2d-d471-4a00-bbea-fe1e4a44aa4f 1.png]]![[221ad4a3-e48d-4f76-95dc-3e3184fffc52.png]]
# Lab
# Create Rolling Update Deployment YAML

```bash
kubectl create deployment rolling-demo \
  --image=nginx:1.24 \
  --replicas=4 \
  -o yaml \
  --dry-run=client > deployment.yaml
```

## Check the File

```bash
ls
```

Output:

```text
deployment.yaml
```

## Edit the File

```bash
vi deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: rolling-demo
  name: rolling-demo
spec:
  replicas: 4
  selector:
    matchLabels:
      app: rolling-demo
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: rolling-demo
    spec:
      containers:
      - image: nginx:1.24
        name: nginx
        resources: {}
status: {}
```

# Apply Rolling Update Deployment

```bash
kubectl apply -f deployment.yaml
```

Output:

```text
deployment.apps/rolling-demo created
```
# Check Rolling Update Pods

```bash
kubectl get pods
```

Output:

```text
NAME                            READY   STATUS    RESTARTS   AGE
rolling-demo-79d7f46598-92m76   1/1     Running   0          3m18s
rolling-demo-79d7f46598-bbcq9   1/1     Running   0          3m18s
rolling-demo-79d7f46598-p5rpz   1/1     Running   0          3m18s
rolling-demo-79d7f46598-rz7z6   1/1     Running   0          3m18s
```
# Watch ReplicaSets in Another Tab

```bash
watch kubectl get rs
```
# Update Deployment Image

```bash
kubectl set image deployment/rolling-demo nginx=nginx:1.25
```

Output:

```text
deployment.apps/rolling-demo image updated
```
# Create Recreate Deployment YAML

```bash
kubectl create deployment recreate-demo \
  --image=nginx:1.24 \
  --replicas=4 \
  -o yaml \
  --dry-run=client > deployment2.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: recreate-demo
  name: recreate-demo
spec:
  replicas: 4
  selector:
    matchLabels:
      app: recreate-demo
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: recreate-demo
    spec:
      containers:
      - image: nginx:1.24
        name: nginx
        resources: {}
status: {}
```
# Apply Recreate Deployment

```bash
kubectl apply -f deployment2.yaml
```

Output:

```text
deployment.apps/recreate-demo created
```

# Update Recreate Deployment Image

```bash
kubectl set image deployment/recreate-demo nginx=nginx:1.25
```

Output:

```text
deployment.apps/recreate-demo image updated
```
![[435a5935-241f-493e-ae30-07eb42fc0a0d.png]]

# ReplicaSet Scaling vs Deployment Strategy

No — adding Pods in a ReplicaSet is **not considered a deployment strategy** by itself.

In Kubernetes:

- A **ReplicaSet (RS)** is responsible for maintaining the desired number of Pods.
- If you increase replicas from `3 → 5`, the ReplicaSet simply creates `2` more Pods.

## Example

```yaml
replicas: 5
```

If currently:

```text
3 Pods running
```

ReplicaSet adds:

```text
+2 new Pods
```

This process is called:

- Scaling
- Scale Up

It is **NOT** a deployment strategy.

---

# Difference

## ReplicaSet Scaling

```text
3 Pods → 5 Pods
```

### Purpose

- Increase capacity
- Handle more traffic
- High availability

No image/version change happens.

---

## Deployment Strategy

Deployment strategies happen when updating the application image/version.

### Example

```text
nginx:1.24 → nginx:1.25
```

### Strategies

- RollingUpdate
- Recreate

These strategies control:

- How old Pods are replaced
- Downtime behavior
- Availability during updates

---

# Simple Summary

| Action               | Is it Strategy? |
| -------------------- | --------------- |
| Add more Pods        | ❌ No            |
| Remove Pods          | ❌ No            |
| Change image/version | ✅ Yes           |
| RollingUpdate        | ✅ Strategy      |
| Recreate             | ✅ Strategy      |

---

# Examples

## Scaling

```bash
kubectl scale deployment myapp --replicas=5
```

Only adds Pods.

---

## Strategy Update

```bash
kubectl set image deployment/myapp nginx=nginx:1.25
```

This triggers:

- RollingUpdate
or
- Recreate strategy




# Lab: RollingUpdate Strategy

## Create Deployment

```bash
kubectl create deployment rolling-demo   --image=nginx:1.24   --replicas=4   -o yaml   --dry-run=client > deployment.yaml
```

---

## Edit Deployment Strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

---

## Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

---

## Update Image

```bash
kubectl set image deployment/rolling-demo nginx=nginx:1.25
```

![[22af3eb4-075e-4799-a994-a4786fd78183.png]]![[9195d255-242f-4fb1-893a-db1b549ec036.png]]

# Kubernetes Requests and Limits

In Kubernetes (k8s), **requests** and **limits** define how much CPU and memory a container can use.

---

# Requests

A **request** is the minimum amount of resources guaranteed for a container.

The Kubernetes scheduler uses this value to decide which node can run the pod.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
```

Meaning:
- CPU: guaranteed 0.25 CPU core
- Memory: guaranteed 256 MiB RAM

---

# Limits

A **limit** is the maximum amount of resources a container can use.

Example:

```yaml
resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Meaning:
- CPU can burst up to 0.5 core
- Memory cannot exceed 512 MiB

---

# Important Behavior

## CPU
- If a container exceeds its CPU limit, it gets throttled
- It is usually not killed

## Memory
- If a container exceeds its memory limit, it may be OOMKilled (Out Of Memory kill)

---

# Common Pattern

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

This means:
- guaranteed: 250m CPU + 256Mi memory
- allowed maximum: 500m CPU + 512Mi memory

---

# Units

## CPU
- `1000m` = 1 CPU core
- `500m` = 0.5 core
- `250m` = 0.25 core

## Memory
- `Mi` = Mebibytes
- `Gi` = Gibibytes

Examples:
- `256Mi`
- `1Gi`

---

# Why Requests and Limits Matter

They help:
- prevent one pod from consuming everything
- improve cluster scheduling
- reduce node instability
- control costs in cloud environments

---

# Full Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo

spec:
  containers:
    - name: app
      image: nginx

      resources:
        requests:
          cpu: "200m"
          memory: "128Mi"

        limits:
          cpu: "1"
          memory: "512Mi"
```

Here:
- pod is guaranteed 0.2 CPU + 128Mi RAM
- pod can use up to 1 CPU + 512Mi RAM


# Lab 
```bash
kubectl get nodes -o wide
```

```bash
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   12d   v1.35.1   172.30.1.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-110-generic   containerd://2.2.1
node01         Ready    <none>          12d   v1.35.1   172.30.2.2    <none>        Ubuntu 24.04.4 LTS   6.8.0-110-generic   containerd://2.2.1
```

```bash
kubectl describe node | grep -A10 "Capacity"
```

```bash
Capacity:
  cpu:                1
  ephemeral-storage:  19221248Ki
  hugepages-2Mi:      0
  memory:             2300156Ki
  pods:               110

Allocatable:
  cpu:                1
  ephemeral-storage:  18698430040
  hugepages-2Mi:      0
  memory:             2197756Ki

--

Capacity:
  cpu:                1
  ephemeral-storage:  19221248Ki
  hugepages-2Mi:      0
  memory:             1948924Ki
  pods:               110

Allocatable:
  cpu:                1
  ephemeral-storage:  18698430040
  hugepages-2Mi:      0
  memory:             1846524Ki
```

```bash
kubectl create deployment nginx-test --image nginx -o yaml --dry-run=client > deployment.yaml
```






