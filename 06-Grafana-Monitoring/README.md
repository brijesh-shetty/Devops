# Lab 06: Real-Time Operations Monitoring with Prometheus & Grafana

## Objective

Create a real-time monitoring application for a delivery service using Python, Prometheus, and Grafana. Simulate delivery metrics, visualize them on dashboards, and set up automated alerts.

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
├── prometheus.yml         # Prometheus configuration
├── alert_rules.yml        # Alert rules configuration
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
- `delivery_service` → scrapes Python app at `172.17.0.1:8000`

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

- **URL:** `http://172.17.0.1:9090`
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

---

## Expected Outputs

- **Metrics Endpoint:** [http://localhost:8000/metrics](http://localhost:8000/metrics)
- **Prometheus Queries:** Graphs for `total_deliveries`, `pending_deliveries`, `average_delivery_time`
- **Grafana Dashboard:** Visualizations for delivery metrics and trends
- **Prometheus Alerts:** Warning/critical alerts for high pending deliveries

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
