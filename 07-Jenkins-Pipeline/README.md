# Lab 07: Jenkins CI/CD Pipeline for Monitoring Stack

## Objective

Automate the deployment of a real-time delivery monitoring stack (Python metrics app, Prometheus, Grafana) using a Jenkins CI/CD pipeline.

---

## Environment Setup

* **OS:** Windows 11 / WSL2 (Ubuntu)
* **Python:** 3.10+
* **Docker:** Docker Desktop
* **Tools:** Jenkins, Prometheus, Grafana

---

## Prerequisites

```bash
pip3 install prometheus-client
```

---

## Directory Structure

```
07-Jenkins-Pipeline/
├── delivery_metrics.py    # Python script to simulate metrics
├── prometheus.yml         # Prometheus configuration
├── alert_rules.yml        # Alert rules configuration
├── Dockerfile             # Dockerfile for the metrics app
├── Jenkinsfile            # Jenkins pipeline script
└── README.md
```

---

## Step-by-Step Execution

### Step 1: Start Jenkins with Docker

Check for existing Jenkins instances:

```bash
docker ps -a | grep jenkins
```

Remove if needed:

```bash
docker rm <container-id>
```

Start a fresh Jenkins instance:

```bash
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins \
  -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

Verify:

```bash
docker ps -a | grep jenkins
```

### Step 2: Get Jenkins Initial Admin Password

```bash
docker exec -it jenkins bash
cat /var/jenkins_home/secrets/initialAdminPassword
exit
```

Access Jenkins at: [http://localhost:8080](http://localhost:8080)

### Step 3: Create a Pipeline Job

1. Go to **New Item → Pipeline → Pipeline script**
2. Paste the contents of `Jenkinsfile` into the **Pipeline Script** section
3. **Save** and **Build Now**

> ⚠️ Update the `localSourcePath` variable in the Jenkinsfile to point to your actual file directory.

### Step 4: Pipeline Stages

The Jenkinsfile defines 5 stages:

| Stage | Description |
|-------|-------------|
| **Pre-check Docker** | Verifies Docker is installed and daemon is running |
| **Setup Workspace** | Copies config files into Jenkins workspace |
| **Build Docker Image** | Builds the `delivery_metrics` Docker image |
| **Run Application** | Runs the metrics app container on port 8000 |
| **Run Prometheus & Grafana** | Starts Prometheus (9090) and Grafana (3000) containers |

### Step 5: Verify the Stack

- **Metrics:** [http://localhost:8000/metrics](http://localhost:8000/metrics)
- **Prometheus:** [http://localhost:9090](http://localhost:9090)
- **Grafana:** [http://localhost:3000](http://localhost:3000) (admin/admin)

### Step 6: Configure Grafana Dashboard

1. Add Prometheus data source: **URL:** `http://172.17.0.1:9090`
2. Create dashboard panels:
   - Total Deliveries (`total_deliveries`)
   - Pending Deliveries (`pending_deliveries`)
   - On-the-Way Deliveries (`on_the_way_deliveries`)
   - Average Delivery Time (`average_delivery_time_sum / average_delivery_time_count`)

### Step 7: Simulate Alerts

Modify `delivery_metrics.py`:

```python
pending = random.randint(50, 100)  # Increase range to exceed alert threshold
```

Restart the app and verify alerts in Prometheus UI and Grafana.

---

## Expected Outputs

- **Jenkins Logs:** Successful pipeline build with logs for each Docker command
- **Metrics Endpoint:** [http://localhost:8000/metrics](http://localhost:8000/metrics)
- **Prometheus Queries:** Graphs for `total_deliveries`, `pending_deliveries`, `average_delivery_time`
- **Grafana Dashboard:** Visualizations for delivery metrics and trends
- **Prometheus Alerts:** Critical alerts for high pending deliveries

---

## Deployment Evidence

<!-- Add your screenshots here -->

---

## Files

| File | Description |
|------|-------------|
| `delivery_metrics.py` | Python script simulating delivery metrics |
| `prometheus.yml` | Prometheus scrape configuration |
| `alert_rules.yml` | Prometheus alerting rules |
| `Dockerfile` | Container image build for metrics app |
| `Jenkinsfile` | Jenkins CI/CD pipeline script |
