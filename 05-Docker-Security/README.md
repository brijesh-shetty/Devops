# Lab 05: Docker Security with AppArmor and Python

## Objective

Secured a Docker container running a Python Flask application using an AppArmor profile. Applied the profile to restrict access to sensitive directories, prevent unauthorized binary execution, and enforce capability restrictions. Verified restrictions using the Docker SDK for Python.

---

## Scenario

Deployed a Python-based web application using Docker with the following security requirements enforced via AppArmor:
- Restrict read access to `/etc/`
- Restrict read/write access to `/var/`
- Prevent execution of binaries in `/bin/` and `/usr/bin/`
- Restrict `sys_admin` capability

---

## Environment Setup

* **OS:** Ubuntu (WSL2)
* **Container Runtime:** Docker Desktop
* **AppArmor Utilities:** `apparmor-utils`
* **Python Docker SDK:** `docker`

---

## Directory Structure

```
05-Docker-Security/
├── app.py                      # Flask web application
├── Dockerfile                  # Container image build instructions
├── my-apparmor-profile         # AppArmor security profile
├── apply_apparmor.py           # Script to apply AppArmor via Docker SDK
├── test_restricted_actions.py  # Script to test restricted actions in container
├── screenshot/                 # Deployment and verification screenshots
└── README.md
```

---

## Prerequisites Installed

```bash
sudo apt-get update
sudo apt-get install apparmor-utils
pip install docker
```

---

## Steps Executed

### Task 1: Wrote the Flask Application (`app.py`)

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, this is a secure Flask application running inside a Docker container!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Task 2: Containerized the Flask Application

`Dockerfile`:

```dockerfile
FROM python:3.8-slim
WORKDIR /app
COPY . /app
RUN pip install flask
EXPOSE 5000
CMD ["python", "app.py"]
```

Built the Docker image:

```bash
docker build -t flask-apparmor .
```

### Task 3: Created and Applied AppArmor Profile

`my-apparmor-profile` (saved to `/etc/apparmor.d/my-apparmor-profile`):

```
#include <tunables/global>

/usr/bin/python3 {
    deny /etc/** r,
    deny /var/** rw,
    network inet stream,
    /app/** rwk,
    deny /bin/** rmix,
    deny /usr/bin/** rmix,
    capability net_bind_service,
    deny capability sys_admin,
}
```

Loaded the profile:

```bash
sudo cp my-apparmor-profile /etc/apparmor.d/my-apparmor-profile
sudo apparmor_parser -r /etc/apparmor.d/my-apparmor-profile
```

Ran the container with the AppArmor profile applied:

```bash
docker run --security-opt="apparmor=my-apparmor-profile" -p 5000:5000 flask-apparmor
```

### Task 4: Applied AppArmor Profile via Docker SDK (`apply_apparmor.py`)

```bash
python apply_apparmor.py
```

**Output:**
```
Container started: f8c2a7f9b9b8
AppArmor profile applied: ['apparmor=my-apparmor-profile']
```

### Task 5: Tested Restricted Actions (`test_restricted_actions.py`)

```bash
python test_restricted_actions.py
```

**Output:**
```
Attempt to read /etc/passwd: Exit Code 1, Output:
Attempt to execute /bin/bash: Exit Code 126, Output:
```

✅ AppArmor profile successfully blocked access to restricted paths and binaries.

---

## Deployment Evidence

![AppArmor Security Screenshot](./screenshot/apparmor-security.png)

---

## Q&A

**Q1. What is the purpose of using AppArmor with Docker containers?**
> AppArmor enforces security policies that confine applications to a limited set of resources, providing an additional layer of security for containers.

**Q2. How do AppArmor profiles help secure a Docker container?**
> They define what a containerized application can or cannot do — restricting access to sensitive directories, network capabilities, file execution, and system calls.

**Q3. Why is it important to restrict access to `/etc/` and `/var/`?**
> These directories contain configuration files, user data, and system settings. Restricting access prevents the container from reading or modifying important system files.

**Q4. What other capabilities can you restrict using AppArmor profiles?**
> Network access, port binding, binary execution, directory writes, and system administration capabilities (`cap_sys_admin`).

**Q5. How can you verify if an AppArmor profile is successfully applied?**
> Inspect the container using Docker SDK or CLI. The `HostConfig.SecurityOpt` field shows the applied security options including the AppArmor profile name.

---

## Files

| File | Description |
|------|-------------|
| `app.py` | Flask web application |
| `Dockerfile` | Container image build instructions |
| `my-apparmor-profile` | AppArmor security profile |
| `apply_apparmor.py` | Script to apply AppArmor via Docker SDK |
| `test_restricted_actions.py` | Script to test restricted actions in the container |
