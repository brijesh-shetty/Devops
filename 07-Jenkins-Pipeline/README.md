# Lab 07: Introduction to Continuous Integration (CI) & Jenkins Pipeline for Monitoring Stack

---

## Part 1: Introduction to Continuous Integration (CI) and Jenkins

### What is Continuous Integration (CI)?
Continuous Integration (CI) is a software engineering practice where developers frequently integrate their code changes into a central repository, usually multiple times a day. Each integration is automatically verified by automated builds and tests, allowing teams to detect and resolve defects early, improve software quality, and accelerate release cycles.

---

### Key Features of CI
1. **Frequent Code Integration**: Developers commit changes to a shared mainline repository regularly.
2. **Automated Builds**: Every commit triggers an automated build process to compile code and resolve dependencies.
3. **Automated Testing**: Unit, integration, and security tests run automatically against each build.
4. **Immediate Feedback**: Developers receive instantaneous notifications on build or test status.

---

### Benefits of CI
- **Early Defect Detection**: Bugs are caught immediately when code is merged, drastically reducing remediation costs.
- **Improved Team Collaboration**: Prevents "integration hell" by keeping team members aligned on the latest code.
- **Faster Delivery Cycles**: Automated validation reduces release friction and accelerates time-to-market.
- **Consistent Code Quality**: Continuous regression testing ensures overall system stability and code standard compliance.

---

### How CI Works
```
+---------------+      Push Code      +-------------------+      Trigger Build      +------------------+
|   Developer   | ------------------> |  Shared Repo/Git  | ----------------------> |    CI Server     |
+---------------+                     +-------------------+                         +------------------+
                                                                                             |
                                                                                    Compiles & Runs Tests
                                                                                             |
                                                                                             v
+---------------+                   Send Build/Test Feedback                        +------------------+
|   Developer   | <---------------------------------------------------------------- |   Test Results   |
+---------------+                                                                   +------------------+
```

1. **Developer Workflow**: Developer completes local work and pushes commits to Git.
2. **CI Server Detection**: The CI server detects changes via webhooks or polling.
3. **Automated Build**: Compiles code, fetches libraries, and packages artifacts.
4. **Automated Testing**: Runs unit, integration, and static code analysis suites.
5. **Feedback**: Results are immediately reported back (e.g., Slack, email, PR comments).

---

### Common CI/CD Tools Overview

| Tool | Overview | Key Highlights | Best Suited For |
|---|---|---|---|
| **GitLab CI/CD** | Fully integrated CI/CD inside GitLab platform | YAML-based (`.gitlab-ci.yml`), Auto DevOps, native registry | Teams utilizing GitLab ecosystem |
| **CircleCI** | Cloud & self-hosted CI with high performance | Container workflows, fast caching, parallel test execution | Container-heavy microservice pipelines |
| **Travis CI** | Cloud-hosted CI widely used in open source | Simple `.travis.yml`, multi-language matrix testing | Open-source libraries and small teams |
| **Atlassian Bamboo** | Enterprise CI/CD tightly bound to Atlassian suite | Direct integration with Jira, Bitbucket, Confluence | Organizations using the Atlassian stack |
| **TeamCity** | Feature-rich CI server by JetBrains | Rich build chains, pre-tested commits, deep IDE integration | Complex enterprise builds |
| **Azure Pipelines** | Cloud CI/CD solution by Microsoft | Multi-platform agents (Linux, macOS, Windows), Azure native | Enterprise and Azure-based projects |
| **GitHub Actions** | Built-in workflow automation directly in GitHub | YAML workflows, extensive marketplace of shared actions | Repositories hosted on GitHub |
| **Spinnaker** | Multi-cloud CD platform originated at Netflix | Canary analysis, automated rollbacks, blue-green deploys | Complex multi-cloud CD deployments |
| **Buildkite** | Hybrid CI platform: cloud UI + self-hosted runners | Scalable local compute, high security for private code | Teams needing full runner infrastructure control |
| **Drone** | Lightweight container-native CI server | Every step is a Docker container, minimal resource overhead | Container-first architectures |

---

### Introduction to Jenkins

#### What is Jenkins?
Jenkins is the industry-standard, open-source automation server for building, testing, and deploying software. It forms the backbone of CI/CD by automating tasks across heterogeneous environments via thousands of community plugins.

#### Core Jenkins Concepts
- **Job / Project**: A user-configured automated task (e.g., Freestyle project, Pipeline).
- **Build**: The single execution of a job or pipeline run.
- **Pipeline**: A workflow definition implemented in code via a `Jenkinsfile` (Declarative or Scripted).
- **Plugins**: Modular extensions that integrate Jenkins with external tools (Git, Docker, Kubernetes, Slack, etc.).
- **Nodes & Agents**: Execution machines where Jenkins distributes and runs job workloads (Master/Agent topology).

---

## Part 2: Lab 07 — Real-Time Operations Monitoring and Alerting Pipeline

### Objective
As a DevOps engineer managing a fast-paced delivery service, build an automated CI/CD pipeline using **Jenkins** that:
1. Builds and containerizes the Python metrics simulation application.
2. Deploys the delivery metrics application container.
3. Provisions **Prometheus** for automated metrics scraping and alerting rules evaluation.
4. Provisions **Grafana** for operational dashboards and real-time visualization.
5. Validates automated alert firing when delivery thresholds are exceeded.

---

### Environment Setup & Prerequisites

- **OS:** Windows 11 / WSL2 / Linux
- **Python:** 3.10+
- **Docker:** Docker Desktop running
- **Tools:** Jenkins, Prometheus, Grafana

#### Install Python Dependencies:
```bash
pip install prometheus-client
```

---

### Directory Structure

```
07-Jenkins-Pipeline/
├── delivery_metrics.py   # Python script to simulate delivery metrics
├── prometheus.yml        # Prometheus scrape configuration
├── alert_rules.yml       # Prometheus alerting rules
├── Dockerfile            # Container image build for metrics application
├── Jenkinsfile           # Jenkins CI/CD declarative pipeline script
├── screenshot/           # Deployment and verification screenshots
└── README.md
```

---

### Step-by-Step Execution

#### Step 1: Start Jenkins with Docker

Check if a Jenkins container is already running:

*On Linux/WSL:*
```bash
docker ps -a | grep jenkins
```

*On Windows PowerShell:*
```powershell
docker ps -a | Select-String "jenkins"
```

If an existing or exited Jenkins container exists, remove it:
```powershell
docker rm -f jenkins
```

Start a fresh Jenkins instance:
```powershell
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins -v jenkins_home:/var/jenkins_home -v //var/run/docker.sock:/var/run/docker.sock -u root jenkins/jenkins:lts
```

> [!NOTE]
> - `-p 8080:8080`: Exposes the web UI on host port 8080.
> - `-p 50000:50000`: Exposes the agent communication port.
> - `-v //var/run/docker.sock:/var/run/docker.sock -u root`: Allows Jenkins to execute Docker commands on the host machine.

---

#### Step 2: Unlock and Set Up Jenkins

1. Retrieve the initial administrator password:
   ```powershell
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
2. Navigate to [http://localhost:8080](http://localhost:8080) in your browser.
3. Paste the initial administrator password and click **Continue**.
4. Select **Install suggested plugins**.
5. Create your initial admin user credentials or continue as admin.

---

#### Step 3: Application & Configuration Files

##### 1. Python Metrics Application (`delivery_metrics.py`)
```python
from prometheus_client import start_http_server, Summary, Gauge
import random
import time

# Metrics
total_deliveries = Gauge("total_deliveries", "Total number of deliveries")
pending_deliveries = Gauge("pending_deliveries", "Number of pending deliveries")
on_the_way_deliveries = Gauge("on_the_way_deliveries", "Number of deliveries on the way")
average_delivery_time = Summary("average_delivery_time", "Average delivery time in seconds")

def simulate_delivery():
    pending = random.randint(10, 20)  # Exceeds threshold to trigger alerts
    on_the_way = random.randint(5, 20)
    delivered = random.randint(30, 70)
    avg_time = random.uniform(15, 45)

    total = pending + on_the_way + delivered

    print(f"[DEBUG] Total deliveries: {total}")
    print(f"[DEBUG] Pending deliveries: {pending}")
    print(f"[DEBUG] On-the-way deliveries: {on_the_way}")
    print(f"[DEBUG] Average delivery time: {avg_time:.2f} seconds")

    total_deliveries.set(total)
    pending_deliveries.set(pending)
    on_the_way_deliveries.set(on_the_way)
    average_delivery_time.observe(avg_time)

if __name__ == "__main__":
    print("[INFO] Starting the HTTP server on port 8000...")
    start_http_server(8000, addr="0.0.0.0")
    print("[INFO] HTTP server started. Simulating deliveries...")
    while True:
        simulate_delivery()
        print("[INFO] Sleeping for 1 seconds...")
        time.sleep(1)
```

##### 2. Prometheus Configuration (`prometheus.yml`)
```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "delivery_service"
    static_configs:
      - targets: ["host.docker.internal:8000"]

rule_files:
  - /etc/prometheus/alert_rules.yml
```

> [!TIP]
> Use `host.docker.internal:8000` when running Docker Desktop on Windows/macOS so containers can reach host-mapped services. On native Linux host networking, use `172.17.0.1:8000`.

##### 3. Alert Rules (`alert_rules.yml`)
```yaml
groups:
  - name: delivery_alerts
    rules:
      - alert: HighPendingDeliveries
        expr: pending_deliveries > 10
        for: 15s
        labels:
          severity: warning
        annotations:
          summary: "High pending deliveries"
          description: "Pending deliveries are above 10 for the last 15 second."

      - alert: HighAverageDeliveryTime
        expr: (average_delivery_time_sum / average_delivery_time_count) > 30
        labels:
          severity: critical
        annotations:
          summary: "High average delivery time"
          description: "Average delivery time is above 30 seconds for the last 15 seconds."
```

##### 4. Dockerfile for Metrics Application (`Dockerfile`)
```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY delivery_metrics.py .

RUN pip install --no-cache-dir prometheus-client

EXPOSE 8000

CMD ["python", "delivery_metrics.py"]
```

---

#### Step 4: Jenkins Pipeline Script (`Jenkinsfile`)

```groovy
pipeline {
    agent any

    stages {
        stage('Pre-check Docker') {
            steps {
                script {
                    try {
                        def dockerVersion = sh(script: 'docker --version', returnStdout: true).trim()
                        if (!dockerVersion) {
                            error "Docker is not installed or not in the PATH. Please install Docker."
                        }

                        def dockerInfo = sh(script: 'docker info', returnStatus: true)
                        if (dockerInfo != 0) {
                            error "Docker daemon is not running. Please start the Docker service."
                        }

                        echo "Docker is available and running: ${dockerVersion}"
                    } catch (Exception e) {
                        error "Pre-check failed: ${e.message}"
                    }
                }
            }
        }

        stage('Setup Workspace') {
            steps {
                script {
                    // Update localSourcePath with your actual local repository path
                    def localSourcePath = '/path/to/your/local/files'
                    def workspacePath = env.WORKSPACE

                    sh """
                        cp ${localSourcePath}/delivery_metrics.py ${workspacePath}/
                        cp ${localSourcePath}/prometheus.yml ${workspacePath}/
                        cp ${localSourcePath}/alert_rules.yml ${workspacePath}/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t delivery_metrics .'
            }
        }

        stage('Run Application') {
            steps {
                sh 'docker run -d -p 8000:8000 --name delivery_metrics delivery_metrics'
            }
        }

        stage('Run Prometheus & Grafana') {
            steps {
                sh '''
                    docker run -d --name prometheus -p 9090:9090 \
                        -v $WORKSPACE/prometheus.yml:/etc/prometheus/prometheus.yml \
                        -v $WORKSPACE/alert_rules.yml:/etc/prometheus/alert_rules.yml \
                        prom/prometheus

                    docker run -d --name grafana -p 3000:3000 grafana/grafana
                '''
            }
        }
    }
}
```

---

#### Step 5: Configure and Run Pipeline in Jenkins

1. On Jenkins Dashboard, click **New Item**.
2. Enter item name (e.g., `delivery-monitoring-pipeline`) and select **Pipeline** -> **OK**.
3. Under the **Pipeline** configuration section:
   - **Definition**: Select *Pipeline script*.
   - Paste the contents of `Jenkinsfile` into the code area.
4. Click **Save**.
5. Click **Build Now** to execute the pipeline.
6. Verify all stages turn green (**SUCCESS**):
   - `Pre-check Docker`
   - `Setup Workspace`
   - `Build Docker Image`
   - `Run Application`
   - `Run Prometheus & Grafana`

---

#### Step 6: Verify Monitoring Stack

Once the pipeline completes, verify each service:

| Service | URL | Default Credentials / Purpose |
|---|---|---|
| **Metrics Endpoint** | [http://localhost:8000/metrics](http://localhost:8000/metrics) | Live Prometheus metrics feed |
| **Prometheus Web UI** | [http://localhost:9090](http://localhost:9090) | Targets, queries, and active alerts |
| **Grafana Dashboard** | [http://localhost:3000](http://localhost:3000) | `admin` / `admin` — Visual dashboards |

##### Grafana Dashboard Setup:
1. Add Prometheus Data Source: URL `http://host.docker.internal:9090` -> Click **Save & Test**.
2. Add panels for:
   - **Total Deliveries**: `total_deliveries`
   - **Pending Deliveries**: `pending_deliveries`
   - **On-the-Way Deliveries**: `on_the_way_deliveries`
   - **Average Delivery Time**: `average_delivery_time_sum / average_delivery_time_count`

---

#### Step 7: Alert Verification

1. In Prometheus, navigate to **Alerts** ([http://localhost:9090/alerts](http://localhost:9090/alerts)).
2. Confirm the alerts are listed under `delivery_alerts`:
   - 🔴 **HighPendingDeliveries** (`pending_deliveries > 10`)
   - 🔴 **HighAverageDeliveryTime** (`> 30s`)
3. When the threshold is exceeded for longer than the evaluation period (`15s`), state changes from **Inactive** → **Pending** → **Firing**.

---

## Expected Outputs

1. **Jenkins Console Output**: Clean successful build logs for all Docker pipeline stages.
2. **Metrics Endpoint Output**: Valid HTTP 200 response with Gauge/Summary data on port 8000.
3. **Prometheus Targets**: `delivery_service` target status showing **UP**.
4. **Grafana Dashboards**: Real-time visual graphs updating live as metrics change.
5. **Prometheus Alerts**: Active firing alerts when delivery metrics cross warning/critical thresholds.

---

## Deployment Evidence

*Add your verification screenshots below:*

- **Jenkins Pipeline Success**:
  <!-- ![Jenkins Pipeline](screenshot/jenkins-pipeline-success.png) -->
- **Prometheus Targets Status**:
  <!-- ![Prometheus Targets](screenshot/prometheus-targets.png) -->
- **Prometheus Alerts Firing**:
  <!-- ![Prometheus Alerts](screenshot/prometheus-alerts.png) -->
- **Grafana Delivery Dashboard**:
  <!-- ![Grafana Dashboard](screenshot/grafana-dashboard.png) -->
