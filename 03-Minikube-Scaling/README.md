# Lab 03: Scaling Flask App on Single Node using ReplicaSets

## Real-Life Tech Use Case: E-commerce Flash Sale

During a flash sale on an e-commerce site (like Flipkart's Big Billion Days or Amazon Prime Day):

- A simple Flask service might normally handle **100 requests per minute**
- Suddenly, traffic spikes to **10,000 requests per minute**
- If the app runs on a single Pod, it will **crash under the load**
- Using **ReplicaSets**, the system can scale out to 10 or 20 Pods running the same app, distributing requests among them
- Once the sale ends and traffic returns to normal, Kubernetes can **scale back down** to save resources

## Objective

Demonstrated Kubernetes ReplicaSets by deploying a Flash Sale Flask app, scaling it from 3 to 5 replicas, and observing self-healing behavior when a pod is deleted.

## Key Observations and Learnings

| Concept | Explanation |
|---------|-------------|
| **Pod Distribution** | Each Pod is like an identical worker. Scaling means creating clones of the app |
| **Resiliency** | If one Pod fails, the ReplicaSet automatically creates another, so users don't notice downtime |
| **Efficiency** | Instead of over-provisioning servers, we add Pods when demand spikes and remove them when demand is low |
| **Real-World Scalability** | Exactly how Netflix, YouTube, or Swiggy scale their microservices to handle peak traffic hours |

---

## Directory Structure

```
03-Minikube-Scaling/
├── app.py                      # Flash sale Flask application
├── Dockerfile                  # Container image build instructions
├── flashsale-replicaset.yaml   # Kubernetes ReplicaSet + Service manifest
├── screenshot/                 # Deployment and verification screenshots
└── README.md
```

---

## App Details

The Flash Sale Flask App (`app.py`) exposes three endpoints:

| Endpoint | Description |
|----------|-------------|
| `/` | Welcomes users to the Big Sale, returns pod name and timestamp |
| `/buy` | Simulates a checkout during a flash sale. Assigns a random product or uses `?user=123` query param. Shows which Pod served the request |
| `/health` | For readiness/liveness probes |

---

## Steps Executed

### Step 1: Started Minikube with a Single Node

```bash
minikube stop
minikube delete
minikube start --nodes=1
```

Verified nodes:

```bash
kubectl get nodes
```

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   42s   v1.31.0
```

### Step 2: Built Docker Image using Minikube's Docker Daemon

```bash
eval $(minikube docker-env)
docker build -t flashsale:1.0 .
```

### Step 3: Applied the ReplicaSet Configuration

```bash
kubectl apply -f flashsale-replicaset.yaml
```

```
replicaset.apps/flashsale-rs created
service/flashsale-svc created
```

### Step 4: Verified the ReplicaSet and Pods (Initial — 3 replicas)

```bash
kubectl get rs
```

```
NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   3         3         3       40s
```

```bash
kubectl get pods
```

```
NAME                 READY   STATUS    RESTARTS   AGE
flashsale-rs-8gbfp   1/1     Running   0          3m35s
flashsale-rs-f4gsl   1/1     Running   0          3m35s
flashsale-rs-nb5kl   1/1     Running   0          3m35s
```

### Step 5: Scaled the ReplicaSet to 5 Replicas

```bash
kubectl scale rs flashsale-rs --replicas=5
```

```
replicaset.apps/flashsale-rs scaled
```

Verified:

```bash
kubectl get rs
```

```
NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   5         5         5       7m38s
```

```bash
kubectl get pods
```

```
NAME                 READY   STATUS    RESTARTS   AGE
flashsale-rs-4nr6q   1/1     Running   0          7m55s
flashsale-rs-84v7x   1/1     Running   0          7m55s
flashsale-rs-nsmlx   1/1     Running   0          32s
flashsale-rs-rbwr4   1/1     Running   0          7m55s
flashsale-rs-wfbb4   1/1     Running   0          32s
```

### Step 6: Deleted One Pod — Observed Self-Healing

```bash
kubectl delete pod flashsale-rs-4nr6q
```

```
pod "flashsale-rs-4nr6q" deleted
```

Kubernetes automatically created a replacement pod to maintain 5 replicas:

```bash
kubectl get pods
```

> **Result:** A new pod was created immediately to restore the desired count of 5 replicas.

### Step 7: Verified Pod Distribution Across the Node

```bash
kubectl get pods -o wide
```

```
NAME                 READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
flashsale-rs-4nr6q   1/1     Running   0          10m     10.244.0.9    minikube   <none>           <none>
flashsale-rs-hqtm7   1/1     Running   0          109s    10.244.0.12   minikube   <none>           <none>
flashsale-rs-nsmlx   1/1     Running   0          2m54s   10.244.0.11   minikube   <none>           <none>
flashsale-rs-rbwr4   1/1     Running   0          10m     10.244.0.7    minikube   <none>           <none>
flashsale-rs-wfbb4   1/1     Running   0          2m54s   10.244.0.10   minikube   <none>           <none>
```

All 5 pods ran on the single `minikube` node as expected for a `--nodes=1` cluster.

---

## Deployment Evidence

![ReplicaSet Scaling Screenshot](./screenshot/replicaset-scaling.png)

---

## Q&A

**Q1. What is the initial number of replicas in the ReplicaSet?**
> 3

**Q2. How many pods are running after applying the ReplicaSet configuration?**
> 3

**Q3. What happens when you scale the ReplicaSet to 5 replicas?**
> Kubernetes creates 2 additional pods to meet the desired number of replicas (5). The ReplicaSet now has 5 running pods.

**Q4. What happens when you delete one pod?**
> Kubernetes automatically creates a new pod to replace the deleted one, maintaining the desired number of replicas (5).

**Q5. How does Kubernetes maintain the desired number of replicas?**
> Kubernetes continuously monitors the number of running pods and compares it to the desired number of replicas. If there's a discrepancy, Kubernetes creates or deletes pods to maintain the desired state.

**Q6. How many nodes are running?**
> 1

**Q7. Where are the pods running with respect to nodes?**
> All 5 pods are running on the single `minikube` node since we started with `--nodes=1`.

---

## Files

| File | Description |
|------|-------------|
| `app.py` | Flask application simulating a flash sale |
| `Dockerfile` | Container image build instructions |
| `flashsale-replicaset.yaml` | Kubernetes ReplicaSet + Service manifest |
