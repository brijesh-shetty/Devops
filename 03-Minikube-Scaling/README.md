# Lab 03 – Scaling Flask App on Single Node using ReplicaSets

## Real-Life Tech Use Case: E-commerce Flash Sale

During a flash sale on an e-commerce site (like Flipkart's Big Billion Days or Amazon Prime Day):

- A simple Flask service might normally handle **100 requests per minute**
- Suddenly, traffic spikes to **10,000 requests per minute**
- If the app runs on a single Pod, it will **crash under the load**
- Using **ReplicaSets**, the system can scale out to 10 or 20 Pods running the same app, distributing requests among them
- Once the sale ends and traffic returns to normal, Kubernetes can **scale back down** to save resources

## Objective

- Understand ReplicaSets and Pods
- Scale Flask App deployment
- Observe pod distribution

## Key Observations and Learnings

| Concept | Explanation |
|---------|-------------|
| **Pod Distribution** | Each Pod is like an identical worker. Scaling means creating clones of the app |
| **Resiliency** | If one Pod fails, the ReplicaSet automatically creates another, so users don't notice downtime |
| **Efficiency** | Instead of over-provisioning servers, we add Pods when demand spikes and remove them when demand is low |
| **Real-World Scalability** | Exactly how Netflix, YouTube, or Swiggy scale their microservices to handle peak traffic hours |

---

## App Details

The Flash Sale Flask App (`app.py`) exposes three endpoints:

| Endpoint | Description |
|----------|-------------|
| `/` | Welcomes users to the Big Sale, returns pod name and timestamp |
| `/buy` | Simulates a checkout during a flash sale. Assigns a random product or uses `?user=123` query param. Shows which Pod served the request |
| `/health` | For readiness/liveness probes |

---

## Steps

### Step 1: Clean Up Previous Minikube

If you already have a running cluster, stop and delete it:

```bash
minikube stop
minikube delete
```

### Step 2: Start Minikube with a Single Node

```bash
minikube start --nodes=1
```

Verify nodes:

```bash
kubectl get nodes
```

Expected output:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   42s   v1.31.0
```

### Step 3: Initialize Minikube Docker Environment and Build Image

```bash
minikube docker-env
eval $(minikube docker-env)
docker build -t flashsale:1.0 .
```

### Step 4: Apply the ReplicaSet Configuration

```bash
kubectl apply -f flashsale-replicaset.yaml
```

Expected output:

```
replicaset.apps/flashsale-rs created
service/flashsale-svc created
```

### Step 5: Verify the ReplicaSet and Pods

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

### Step 6: Scale the ReplicaSet to 5 Replicas

```bash
kubectl scale rs flashsale-rs --replicas=5
```

```
replicaset.apps/flashsale-rs scaled
```

### Step 7: Verify the Updated ReplicaSet

```bash
kubectl get rs
```

```
NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   5         5         5       7m38s
```

### Step 8: Verify the Updated Pods

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

### Step 9: Delete One Pod and Observe Self-Healing

```bash
kubectl delete pod <pod-name>
```

```
pod "<pod-name>" deleted
```

Verify that Kubernetes automatically creates a replacement:

```bash
kubectl get pods
```

> **Note:** A new pod appears to maintain the desired count of 5 replicas.

### Step 10: View Pod Distribution Across Nodes

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

All 5 pods are running on the single `minikube` node.

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

## Additional Challenges

- Update the `flashsale-replicaset.yaml` file to use a different image
- Create a **Deployment** instead of a ReplicaSet
- Use `kubectl describe` to inspect the ReplicaSet and pods

## Tips and Variations

- Use `kubectl get pods -o wide` to see pod distribution across nodes
- Use `kubectl logs <pod-name>` to view pod logs
- Use `kubectl exec -it <pod-name> -- /bin/bash` to access a pod's container

---

## Files

| File | Description |
|------|-------------|
| `app.py` | Flask application simulating a flash sale |
| `Dockerfile` | Container image build instructions |
| `flashsale-replicaset.yaml` | Kubernetes ReplicaSet + Service manifest |
