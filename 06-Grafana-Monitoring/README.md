# Lab 06: Real-Time Operations Monitoring with Prometheus & Grafana

## Objective

Built a real-time monitoring application for a ZAPPTTO delivery service using Python, Prometheus, and Grafana. Simulated delivery metrics, visualised them on Grafana dashboards, and configured Prometheus alerting rules that fired when thresholds were exceeded.

---

## Environment Setup

* **OS:** Windows 11 / WSL2 (Ubuntu)
* **Python:** 3.10+
* **Docker:** Docker Desktop
* **Tools:** Prometheus, Grafana

---

## Prerequisites

```bash
pip3 install prometheus-client
```

---

## Directory Structure

```
06-Grafana-Monitoring/
├── delivery_metrics.py    # Python script to simulate metrics
├── prometheus.yml         # Prometheus scrape configuration
├── alert_rules.yml        # Prometheus alerting rules
├── Dockerfile             # Container image for metrics app
├── Jenkinsfile            # Jenkins pipeline to run the full stack
├── screenshot/            # Verification screenshots
└── README.md
```

---

## Step-by-Step Execution

### Step 1: Run the Python Metrics Application

```bash
python3 delivery_metrics.py
```

**Output:**
```
[INFO] Starting the HTTP server on port 8000...
[INFO] HTTP server started. Simulating deliveries...
[DEBUG] Total deliveries: 69
[DEBUG] Pending deliveries: 20
[DEBUG] On-the-way deliveries: 17
[DEBUG] Average delivery time: 28.26 seconds
```

Verify metrics at: [http://localhost:8000/metrics](http://localhost:8000/metrics)

```bash
curl http://localhost:8000/metrics
```

> **Note:** The `/metrics` endpoint is automatically created by the `prometheus_client` library via `start_http_server`. Metric types include Counter, Gauge, Histogram, and Summary.

### Step 2: Configure and Run Prometheus

`prometheus.yml` defines scrape jobs:
- `prometheus` → scrapes itself at `localhost:9090`
- `delivery_service` → scrapes Python app at `host.docker.internal:8000`

> **Note:** `host.docker.internal` resolves to the host machine from inside a Docker container on Windows/macOS Docker Desktop. On native Linux, use `172.17.0.1:8000` instead.

Run Prometheus:

```bash
docker run -d --name prometheus --network=host \
  -v ./prometheus.yml:/etc/prometheus/prometheus.yml \
  -v ./alert_rules.yml:/etc/prometheus/alert_rules.yml \
  prom/prometheus
```

Verify:

```bash
docker ps -a | grep prom
```

Access Prometheus at: [http://localhost:9090](http://localhost:9090)

Verify targets via: **Prometheus → Status → Targets**

### Step 3: Set Up Grafana

Run Grafana:

```bash
docker run -d --name grafana -p 3000:3000 grafana/grafana
```

#### 3a. Access Grafana

Open [http://localhost:3000](http://localhost:3000)

#### 3b. Login

- **Username:** `admin`
- **Password:** `admin`

#### 3c. Add Prometheus as Data Source

Navigate: **Home → Connections → Data Sources → Prometheus**

- **URL:** `http://host.docker.internal:9090`
- Click **Save & Test**

#### 3d. Create Dashboard

Create a new dashboard with panels:

| Panel | Metric Query |
|-------|-------------|
| Total Deliveries | `total_deliveries` |
| Pending Deliveries | `pending_deliveries` |
| On-the-Way Deliveries | `on_the_way_deliveries` |
| Average Delivery Time | `average_delivery_time_sum / average_delivery_time_count` |

### Step 4: Simulate Alerts

Modify `delivery_metrics.py` to increase pending values:

```python
pending = random.randint(50, 100)  # Increase range to exceed alert threshold
```

Restart the Python script and verify alerts in Prometheus UI and Grafana.

### Step 5: Automate with Jenkins (Optional)

A `Jenkinsfile` is provided to automate the entire stack via a Jenkins pipeline:

1. Start Jenkins: `docker run -d -p 8080:8080 --name jenkins -v jenkins_home:/var/jenkins_home -v //var/run/docker.sock:/var/run/docker.sock -u root jenkins/jenkins:lts`
2. Open [http://localhost:8080](http://localhost:8080) and unlock Jenkins.
3. Create a new **Pipeline** job, paste the contents of `Jenkinsfile`, and click **Build Now**.
4. All 5 stages (`Pre-check Docker`, `Setup Workspace`, `Build Docker Image`, `Run Application`, `Run Prometheus & Grafana`) should turn green.

---

## Expected Outputs

- **Metrics Endpoint:** [http://localhost:8000/metrics](http://localhost:8000/metrics)
- **Prometheus Queries:** Graphs for `total_deliveries`, `pending_deliveries`, `average_delivery_time`
- **Grafana Dashboard:** Visualizations for delivery metrics and trends
- **Prometheus Alerts:** Warning/critical alerts for high pending deliveries

---

## Deployment Evidence

![Prometheus Targets](./screenshot/prometheus-targets.png)

![Grafana Dashboard](./screenshot/grafana-dashboard.png)

![Prometheus Alerts Firing](./screenshot/prometheus-alerts.png)

---

## Files

| File | Description |
|------|-------------|
| `delivery_metrics.py` | Python script simulating delivery metrics |
| `prometheus.yml` | Prometheus scrape configuration |
| `alert_rules.yml` | Prometheus alerting rules |
| `Dockerfile` | Containerises the Python metrics app |
| `Jenkinsfile` | Jenkins pipeline for full stack automation |
