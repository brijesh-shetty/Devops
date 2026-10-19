# Lab 04: Docker Networking with Multiple Containers

## Objective

Configured Docker networking to connect a Python Flask web server, MySQL database, and Redis cache over a custom bridge network, demonstrating container-to-container communication by DNS name.

---

## Environment Setup

* **OS:** Windows 11 / WSL2
* **Container Runtime:** Docker Desktop
* **Images:** `python:3.9-slim`, `mysql:latest`, `redis:latest`

---

## Directory Structure

```
04-Docker-Networking/
├── app.py              # Flask REST API application
├── requirements.txt    # Python dependencies
├── Dockerfile          # Container image build instructions
├── screenshot/         # Deployment and verification screenshots
└── README.md
```

---

## Steps Executed

### Task 1: Created a Custom Bridge Network

```bash
docker network create --driver bridge my-bridge-net
```

**Output:**
```
87c23b491f5994c74e497f4f4f4f4f4f4f4f4
```

### Task 2: Verified the Network

```bash
docker network ls
```

**Output:**
```
NETWORK ID     NAME            DRIVER    SCOPE
87c23b491f59   my-bridge-net   bridge    local
```

### Task 3: Inspected the Network

```bash
docker network inspect my-bridge-net
```

**Output:**
```json
[
  {
    "Name": "my-bridge-net",
    "Driver": "bridge",
    "IPAM": {
      "Config": [
        {
          "Subnet": "172.18.0.0/16",
          "Gateway": "172.18.0.1"
        }
      ]
    },
    "Containers": {}
  }
]
```

### Task 4: Built the Flask Image & Launched All Containers

Built the Flask image:

```bash
docker build -t flask-api .
```

Launched all three containers on the same bridge network:

```bash
docker run -d --name mysql --net=my-bridge-net mysql:latest
docker run -d --name redis --net=my-bridge-net redis:latest
docker run -d --name flask --net=my-bridge-net -p 5001:5001 flask-api
```

### Task 5: Tested Container-to-Container Connectivity

Exec'd into the Flask container and pinged other containers by DNS name:

```bash
docker exec -it flask bash
```

```bash
ping mysql
# PING mysql (172.18.0.2) 56(84) bytes of data.
# 64 bytes from mysql (172.18.0.2): icmp_seq=1 ttl=64 time=0.078 ms

ping redis
# PING redis (172.18.0.3) 56(84) bytes of data.
# 64 bytes from redis (172.18.0.3): icmp_seq=1 ttl=64 time=0.078 ms
```

✅ Containers communicate using their names as DNS hostnames on the custom bridge network.

### Task 6: Cleaned Up

```bash
docker stop mysql redis flask
docker rm mysql redis flask
docker network rm my-bridge-net
```

---

## Deployment Evidence

![Docker Networking Screenshot](./screenshot/docker-networking.png)

---

## Q&A

**Q1. What is the purpose of the `--net` flag in `docker run`?**
> The `--net` flag specifies the network to connect the container to.

**Q2. How do containers communicate with each other on the same network?**
> Containers communicate using their container names (DNS) or IP addresses.

**Q3. What is the difference between a bridge network and a host network?**
> - **Bridge Network:** Containers are in a private, isolated network and talk to each other through the Docker bridge. Think of it as containers in a private room, communicating through a door.
> - **Host Network:** Containers share the host's network stack directly. Think of it as containers sitting in the same room as the host, talking directly to everyone.

**Q4. How can you expose a container's port to the host machine?**
> Use the `-p` flag to map a container port to a host port (e.g., `-p 5001:5001`).

---

## Files

| File | Description |
|------|-------------|
| `app.py` | Flask REST API application |
| `requirements.txt` | Python dependencies |
| `Dockerfile` | Container image build instructions |
