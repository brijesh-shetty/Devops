# Lab 02: Deploy a Flask App on Minikube using kubectl and YAML

## Objective

Deployed a Python Flask application on a single-node Minikube Kubernetes cluster using `kubectl` and YAML manifests, demonstrating Kubernetes fundamentals including Deployments, Services, and NodePort access.

---

## Environment Setup

* **OS:** Windows 11 / WSL2
* **Container Runtime:** Docker Desktop
* **Minikube Version:** `v1.38.1`
* **Kubernetes Version:** `v1.35.1`
* **CLI Tools:** `kubectl`, `minikube`

---

## Directory Structure

```
02-Kubectl-Flask/
├── app.py             # Flask application serving on port 15000
├── Dockerfile         # Container image build instructions
├── flask-app.yaml     # Kubernetes Deployment + NodePort Service manifest
├── screenshot/        # Deployment and verification screenshots
└── README.md
```

---

## Minikube Cheat Sheet

```bash
# Starting and Stopping
minikube start                                    # Start with default settings
minikube start --kubernetes-version=v1.21.2       # Start with specific K8s version
minikube stop                                     # Stop without deleting
minikube delete                                   # Delete cluster

# Status and Info
minikube status                                   # Check status
kubectl cluster-info                              # Display cluster info

# Accessing Services
minikube service <service-name>                   # Open service in browser
minikube service list                             # List all service URLs
minikube tunnel                                   # Tunnel for LoadBalancer services

# Docker with Minikube
eval $(minikube docker-env)                       # Use Minikube's Docker daemon
docker build -t my-image .                        # Build image in Minikube
minikube image load my-image                      # Load local image into Minikube

# Debugging
minikube logs                                     # View Minikube logs
minikube dashboard                                # Open K8s dashboard
minikube ssh                                      # SSH into the Minikube VM
```

---

## Step-by-Step Execution

### Step 1: Started Minikube

```bash
minikube start
```

### Step 2: Flask Application (`app.py`)

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Flask on Kubernetes!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=15000)
```

### Step 3: Dockerfile

```dockerfile
FROM python:3.8-slim
WORKDIR /app
COPY . /app
RUN pip install flask
CMD ["python", "app.py"]
```

### Step 4: Built the Docker Image with Minikube's Docker Daemon

```bash
eval $(minikube docker-env)
docker build -t flask-app .
```

> **Note:** `eval $(minikube docker-env)` configures the Docker CLI to use Minikube's Docker daemon so the image is available inside the cluster without a registry.

### Step 5: Kubernetes Deployment YAML (`flask-app.yaml`)

Initial version (Deployment only):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask-app
          image: flask-app:latest
          imagePullPolicy: Never
          ports:
            - containerPort: 15000
```

> **About `imagePullPolicy: Never`:** Tells Kubernetes to only use the local image and never pull from Docker Hub.

### Step 6: Deployed the Application

```bash
kubectl apply -f flask-app.yaml
```

### Step 7: Verified Deployment Status

```bash
kubectl get deployments
```

```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
flask-app   1/1     1            1           5s
```

### Step 8: Verified Pods

```bash
kubectl get pods -l app=flask-app
```

```
NAME                        READY   STATUS    RESTARTS   AGE
flask-app-3443a-213a        1/1     Running   0          5s
```

### Step 9: Discovered Port Access Issue

Without a Service, the app is not reachable externally:

```bash
curl http://127.0.0.1:15000
# curl: (7) Failed to connect to 127.0.0.1 port 15000
```

> **Why it fails:** Minikube runs in a VM. A **Service** of type `NodePort` is required to expose the port externally.

### Step 10: Added NodePort Service to `flask-app.yaml`

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
    - port: 15000
      targetPort: 15000
  type: NodePort
```

Applied the updated manifest:

```bash
kubectl apply -f flask-app.yaml
```

```
deployment.apps/flask-app unchanged
service/flask-app-service created
```

### Step 11: Accessed Flask App via Minikube

```bash
minikube service flask-app-service --url
```

```
http://127.0.0.1:36157
```

Tested with curl:

```bash
curl http://127.0.0.1:36157
```

```
Hello from Flask on Kubernetes!
```

✅ Flask application successfully accessible via Kubernetes NodePort Service.

---

## Key Concepts: Port vs TargetPort

| Concept | Description |
|---------|-------------|
| **port** | The port exposed by the Kubernetes Service (external-facing) |
| **targetPort** | The port the container is listening on (internal) |

**Request flow:**
```
External Request (port 15000) → Service (port 15000) → Container (targetPort 15000)
```

---

## Deployment Evidence

![Flask App on Minikube](./screenshot/flask-minikube.png)

---

## Q&A

**Q1. What is the purpose of `minikube service flask-app-service --url`?**
> To provide the URL for accessing the `flask-app-service` running in Minikube.

**Q2. What happens when you run `minikube service flask-app-service --url`?**
> Minikube checks if the service is running, generates a URL, and displays it in the terminal.

**Q3. Why is `targetPort` used in Kubernetes Service configuration?**
> To specify the port number on which the container is listening.

**Q4. What is the difference between `port` and `targetPort`?**
> `port` is the exposed Service port, while `targetPort` is the container port.

**Q5. How do you access a Flask application running in Minikube?**
> Use `minikube service <service-name> --url` to get the access URL.

**Q6. Why does the terminal need to remain open when using Docker driver on Linux?**
> Because the Docker driver requires the terminal to stay open to maintain the Minikube tunnel connection.

**Q7. What is the benefit of using `--url` flag with `minikube service`?**
> It provides a convenient way to get the access URL without manually constructing it.

**Q8. What command is used to expose a service in Kubernetes?**
> `kubectl expose` or `kubectl apply -f <service.yaml>`.

**Q9. How does Minikube help in local Kubernetes testing?**
> Minikube runs a single-node Kubernetes cluster locally, making it easy to test deployments without a full production cluster.

**Q10. What is the role of `kubectl` in this setup?**
> `kubectl` is the Kubernetes CLI tool used to deploy applications, manage resources, and interact with the cluster.

---

## Files

| File | Description |
|------|-------------|
| `app.py` | Flask application serving on port 15000 |
| `Dockerfile` | Container image build instructions |
| `flask-app.yaml` | Kubernetes Deployment + NodePort Service manifest |
