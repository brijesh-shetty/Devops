# Lab 07: Introduction to Continuous Integration (CI) & Jenkins

---

## Part 1: What is Continuous Integration (CI)?

Continuous Integration (CI) is a software engineering practice where developers frequently integrate their code changes into a central repository — usually multiple times a day. Each integration is automatically verified by automated builds and tests, allowing teams to detect and resolve defects early, improve software quality, and accelerate release cycles.

---

### Key Features of CI

1. **Frequent Code Integration**: Developers commit changes to a shared mainline repository regularly.
2. **Automated Builds**: Every commit triggers an automated build process to compile code and resolve dependencies.
3. **Automated Testing**: Unit, integration, and security tests run automatically against each build.
4. **Immediate Feedback**: Developers receive instantaneous notifications on build or test status.

---

### Benefits of CI

- **Early Defect Detection**: Bugs are caught immediately when code is merged, reducing remediation costs.
- **Improved Team Collaboration**: Prevents "integration hell" by keeping team members aligned on the latest code.
- **Faster Delivery Cycles**: Automated validation reduces release friction and accelerates time-to-market.
- **Consistent Code Quality**: Continuous regression testing ensures overall system stability.

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

1. Developer pushes commits to Git.
2. CI server detects changes via webhooks or polling.
3. Automated build: compiles code, fetches libraries, packages artifacts.
4. Automated testing: runs unit, integration, and static analysis suites.
5. Feedback reported back to developers instantly.

---

## Part 2: Popular CI/CD Tools

| Tool | Overview | Key Highlights | Best Suited For |
|---|---|---|---|
| **GitLab CI/CD** | Fully integrated CI/CD inside GitLab platform | YAML-based (`.gitlab-ci.yml`), Auto DevOps | Teams using GitLab ecosystem |
| **CircleCI** | Cloud & self-hosted CI with high performance | Container workflows, fast caching, parallel execution | Container-heavy microservice pipelines |
| **Travis CI** | Cloud-hosted CI widely used in open source | Simple `.travis.yml`, multi-language matrix testing | Open-source libraries and small teams |
| **Atlassian Bamboo** | Enterprise CI/CD tightly bound to Atlassian suite | Direct integration with Jira, Bitbucket, Confluence | Organizations using the Atlassian stack |
| **TeamCity** | Feature-rich CI server by JetBrains | Rich build chains, pre-tested commits, deep IDE integration | Complex enterprise builds |
| **Azure Pipelines** | Cloud CI/CD solution by Microsoft | Multi-platform agents, Azure native | Enterprise and Azure-based projects |
| **GitHub Actions** | Built-in workflow automation directly in GitHub | YAML workflows, extensive marketplace of shared actions | Repositories hosted on GitHub |
| **Jenkins** | Industry-standard open-source automation server | Thousands of plugins, highly extensible, self-hosted | Any team needing full CI/CD control |

---

## Part 3: Introduction to Jenkins

### What is Jenkins?
Jenkins is the industry-standard, open-source automation server for building, testing, and deploying software. It forms the backbone of CI/CD by automating tasks across heterogeneous environments via thousands of community plugins.

### Core Jenkins Concepts

| Concept | Description |
|---|---|
| **Job / Project** | A user-configured automated task (e.g., Freestyle project, Pipeline) |
| **Build** | A single execution of a job or pipeline run |
| **Pipeline** | A workflow definition implemented in code via a `Jenkinsfile` |
| **Plugins** | Modular extensions integrating Jenkins with Git, Docker, Kubernetes, etc. |
| **Nodes & Agents** | Execution machines where Jenkins distributes and runs job workloads |

---

## Part 4: Lab — Install and Run Jenkins

Installed and ran Jenkins using Docker, unlocked the dashboard, and verified the Jenkins UI was accessible at `http://localhost:8080`.

### Step 1: Start Jenkins with Docker

Check for an existing Jenkins container:

```powershell
docker ps -a | Select-String "jenkins"
```

Remove any old container if present:

```powershell
docker rm -f jenkins
```

Start a fresh Jenkins instance:

```powershell
docker run -d -p 8080:8080 -p 50000:50000 --name jenkins `
  -v jenkins_home:/var/jenkins_home `
  -v //var/run/docker.sock:/var/run/docker.sock `
  -u root jenkins/jenkins:lts
```

> **Note:** `-v //var/run/docker.sock:/var/run/docker.sock -u root` allows Jenkins to run Docker commands on the host machine.

---

### Step 2: Unlock Jenkins

1. Get the initial admin password:
   ```powershell
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
2. Open [http://localhost:8080](http://localhost:8080) in your browser.
3. Paste the password and click **Continue**.
4. Select **Install suggested plugins**.
5. Create your admin user credentials.

---

### Step 3: Verify Jenkins is Running

Confirm the Jenkins dashboard loads at [http://localhost:8080](http://localhost:8080).

Take a screenshot of the Jenkins dashboard as **deployment evidence**.

---

## Directory Structure

```
07-Jenkins-Pipeline/
├── screenshot/    # Screenshot of Jenkins running
└── README.md
```

---

## Deployment Evidence

![Jenkins Dashboard Running](./screenshot/jenkins-dashboard.png)
