# Lab 10: Multi-Node Kubernetes Cluster with Multiple Applications and ReplicaSets

## Objective

Deployed two Python Flask microservices (Product Catalog and Shopping Cart) on a 3-node Minikube Kubernetes cluster using Deployments and ReplicaSets. Configured pod anti-affinity rules to distribute pods across nodes, achieving high availability and fault tolerance.

---

## Scenario — E-Commerce Platform

You are developing an e-commerce platform with the following services:
1. **Product Catalog (AppA)** — Provides information about available products (2 replicas).
2. **Shopping Cart (AppB)** — Manages user shopping carts (3 replicas).

Each replica should be distributed across multiple nodes to handle traffic and provide redundancy. Even if one node fails, the services remain available for customers.

---

## Prerequisites

- **Docker** installed and running
- **Minikube** installed
- **kubectl** configured
- **Python 3** with Flask

---

## Directory Structure

```
10-Kubernetes-MultiNode/
├── product_catalog.py                  # Product Catalog Flask application
├── shopping_cart.py                    # Shopping Cart Flask application
├── Dockerfile.product                  # Dockerfile for Product Catalog
├── Dockerfile.shopping                 # Dockerfile for Shopping Cart
├── product_catalog_deployment.yaml     # K8s Deployment (2 replicas)
├── shopping_cart_deployment.yaml       # K8s Deployment (3 replicas)
├── product_catalog_service.yaml        # K8s NodePort Service
├── shopping_cart_service.yaml          # K8s NodePort Service
├── screenshot/                         # Deployment and verification screenshots
└── README.md
```

---

## Step-by-Step Execution

### Step 0: Clean Up Any Existing Minikube

```bash
minikube stop
minikube delete
```

---

### Step 1a: Set Up Minikube with Multiple Nodes

```bash
minikube start --nodes 3 -p devops-multinode --force
```

This creates a 3-node Minikube cluster in a profile named `devops-multinode`.

> [!NOTE]
> The `--force` flag is used because the Docker driver should not be used with root privileges. Use `--force` to bypass this restriction when necessary.

---

### Step 1b: Enable the Registry Add-on

For multi-node clusters, use the Minikube registry add-on to share images between nodes:

```bash
minikube -p devops-multinode addons enable registry
```

Expected output:
```
💡  registry is an addon maintained by minikube.
📘  For more information see: https://minikube.sigs.k8s.io/docs/drivers/docker
    ▪ Using image gcr.io/k8s-minikube/kube-registry-proxy:0.0.6
    ▪ Using image docker.io/registry:2.8.3
🔎  Verifying registry addon...
🌟  The 'registry' addon is enabled
```

---

### Step 2: Product Catalog Application (`product_catalog.py`)

```python
from flask import Flask, jsonify

app = Flask(__name__)

products = [
    {"id": 1, "name": "Laptop", "price": 1200},
    {"id": 2, "name": "Phone", "price": 800},
    {"id": 3, "name": "Headphones", "price": 150},
]

@app.route("/products", methods=["GET"])
def get_products():
    return jsonify(products)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

---

### Step 3: Shopping Cart Application (`shopping_cart.py`)

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

cart = []

@app.route("/cart", methods=["GET"])
def get_cart():
    return jsonify(cart)

@app.route("/cart", methods=["POST"])
def add_to_cart():
    item = request.json
    cart.append(item)
    return jsonify(cart), 201

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

---

### Step 4: Dockerize Applications

**Product Catalog Dockerfile** (`Dockerfile.product`):

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY product_catalog.py /app/
RUN pip install flask
CMD ["python", "product_catalog.py"]
```

**Shopping Cart Dockerfile** (`Dockerfile.shopping`):

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY shopping_cart.py /app/
RUN pip install flask
CMD ["python", "shopping_cart.py"]
```

Build the Docker images:

```bash
docker build -t product-catalog:latest -f Dockerfile.product .
docker build -t shopping-cart:latest -f Dockerfile.shopping .
```

Verify images:
```bash
docker images
```

Expected output:
```
REPOSITORY              TAG        IMAGE ID       CREATED          SIZE
shopping-cart           latest     cdfedc6f31ff   8 seconds ago    137MB
product-catalog         latest     2e47c49941d7   53 seconds ago   137MB
```

---

### Step 5: Load Images into Minikube Registry

```bash
minikube -p devops-multinode image load product-catalog:latest
minikube -p devops-multinode image load shopping-cart:latest
```

Verify images are loaded:

```bash
minikube -p devops-multinode ssh -- docker images
```

---

### Step 6: Deploy Applications on Kubernetes

**Product Catalog Deployment** (2 replicas with anti-affinity):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-catalog
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: product-catalog
  template:
    metadata:
      labels:
        app: product-catalog
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: product-catalog
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: product-catalog-container
        image: product-catalog:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
```

**Shopping Cart Deployment** (3 replicas with anti-affinity):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shopping-cart
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: shopping-cart
  template:
    metadata:
      labels:
        app: shopping-cart
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: shopping-cart
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: shopping-cart-container
        image: shopping-cart:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
```

Apply the deployments:

```bash
kubectl apply -f product_catalog_deployment.yaml
kubectl apply -f shopping_cart_deployment.yaml
```

---

### Step 7: Expose the Services

Apply the service configurations:

```bash
kubectl apply -f product_catalog_service.yaml
kubectl apply -f shopping_cart_service.yaml
```

---

### Step 8: Verify Deployment and Distribution

```bash
kubectl get pods -o wide
```

Expected output:
```
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
product-catalog-5fdfc7f5fb-2jlnv   1/1     Running   0          29s   10.244.1.3   devops-multinode-m02   <none>           <none>
product-catalog-5fdfc7f5fb-7v556   1/1     Running   0          29s   10.244.2.4   devops-multinode-m03   <none>           <none>
shopping-cart-788c48675b-7l4lx     1/1     Running   0          25s   10.244.0.4   devops-multinode       <none>           <none>
shopping-cart-788c48675b-d48w2     1/1     Running   0          25s   10.244.1.4   devops-multinode-m02   <none>           <none>
shopping-cart-788c48675b-hml9r     1/1     Running   0          25s   10.244.2.5   devops-multinode-m03   <none>           <none>
```

Each pod for Product Catalog and Shopping Cart should be running on different nodes.

---

### Step 9: Access the Services

#### 9a: Access Product Catalog Service

Open a new terminal:

```bash
minikube -p devops-multinode service product-catalog-service
```

Note the URL (e.g., `http://127.0.0.1:35855`).

#### 9b: Access Shopping Cart Service

Open another terminal:

```bash
minikube -p devops-multinode service shopping-cart-service
```

Note the URL (e.g., `http://127.0.0.1:38975`).

---

### Step 10: Testing with `curl`

#### Test Product Catalog Service

```bash
curl http://127.0.0.1:35855/products
```

Expected response:
```json
[
    {"id": 1, "name": "Laptop", "price": 1200},
    {"id": 2, "name": "Phone", "price": 800},
    {"id": 3, "name": "Headphones", "price": 150}
]
```

#### Test Shopping Cart Service

1. **Retrieve empty cart:**
   ```bash
   curl http://127.0.0.1:38975/cart
   ```
   Expected response:
   ```json
   []
   ```

2. **Add item to cart:**
   ```bash
   curl -X POST http://127.0.0.1:38975/cart -H "Content-Type: application/json" -d '{"id": 1, "name": "Laptop", "quantity": 1}'
   ```
   Expected response:
   ```json
   [
       {"id": 1, "name": "Laptop", "quantity": 1}
   ]
   ```

---

## Summary

In this exercise, we deployed two Python applications (Product Catalog and Shopping Cart) on a multi-node Minikube cluster with ReplicaSets, ensuring each pod runs on a different node. This setup maximizes availability and fault tolerance across the cluster.

---

## Deployment Evidence

![Minikube Multi-Node Status](./screenshot/minikube-nodes.png)

![Pod Distribution Across Nodes](./screenshot/pod-distribution.png)

![Product Catalog API Response](./screenshot/product-catalog-api.png)

![Shopping Cart API Response](./screenshot/shopping-cart-api.png)

---

## Files

| File | Description |
|------|-------------|
| `product_catalog.py` | Product Catalog Flask application |
| `shopping_cart.py` | Shopping Cart Flask application |
| `Dockerfile.product` | Dockerfile for Product Catalog service |
| `Dockerfile.shopping` | Dockerfile for Shopping Cart service |
| `product_catalog_deployment.yaml` | Kubernetes Deployment (2 replicas, anti-affinity) |
| `shopping_cart_deployment.yaml` | Kubernetes Deployment (3 replicas, anti-affinity) |
| `product_catalog_service.yaml` | Kubernetes NodePort Service for Product Catalog |
| `shopping_cart_service.yaml` | Kubernetes NodePort Service for Shopping Cart |
