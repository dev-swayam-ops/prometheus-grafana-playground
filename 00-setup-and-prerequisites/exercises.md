# Exercises: Setup and Prerequisites

## Exercise 1: Docker Installation Verification (Easy)
Install Docker and verify the version is 20.10 or higher.

**Task**:
```bash
docker --version
docker run hello-world
```

**Validation**: Both commands complete without errors.

---

## Exercise 2: Docker Compose Setup (Easy)
Install Docker Compose and confirm it's working.

**Task**:
```bash
docker-compose --version
```

**Validation**: Output shows version 1.29 or higher.

---

## Exercise 3: Clone Repository (Easy)
Clone the prometheus-grafana-playground repository and verify folder structure.

**Task**:
```bash
git clone <repo-url>
cd prometheus-grafana-playground
ls -la
```

**Validation**: All module folders are visible (00-*, 01-*, etc.).

---

## Exercise 4: Docker Daemon Status (Easy)
Check if Docker daemon is running on your system.

**Task**:
```bash
docker ps
docker info | grep "Containers"
```

**Validation**: No "Cannot connect" errors; container count is displayed.

---

## Exercise 5: System Resource Check (Medium)
Verify your system has minimum resources (4GB RAM, 20GB disk).

**Task**:
```bash
# Linux/Mac
free -h
df -h

# Windows (PowerShell)
wmic logicaldisk get size,freespace
wmic os get totalvirtualmemory,totalvisiblememorysise
```

**Validation**: RAM ≥ 4GB, disk free ≥ 20GB.

---

## Exercise 6: Docker Network Test (Medium)
Create a simple Docker network and run two containers to test connectivity.

**Task**:
```bash
docker network create test-network
docker run --rm --name container1 --network test-network alpine ping -c 2 container2
docker run --rm --name container2 --network test-network alpine echo "Container 2 running"
```

**Validation**: Containers communicate without connection errors.

---

## Exercise 7: Docker Volume Practice (Medium)
Create a Docker volume and mount it to a container.

**Task**:
```bash
docker volume create test-volume
docker run --rm -v test-volume:/data alpine echo "data" > /data/test.txt
docker volume inspect test-volume
```

**Validation**: Volume is created and mounted successfully.

---

## Exercise 8: Git Configuration (Medium)
Configure Git with your identity and verify settings.

**Task**:
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global --list
```

**Validation**: User name and email appear in config output.

---

## Exercise 9: Docker Cleanup (Medium)
Clean up unused Docker resources and verify disk space.

**Task**:
```bash
docker system prune -a --volumes --force
docker images
docker ps -a
df -h
```

**Validation**: No unused containers/images remain; disk space increased.

---

## Exercise 10: Environment Variable Export (Medium)
Set environment variables for your monitoring setup.

**Task**:
```bash
# Linux/Mac
export DOCKER_HOST=unix:///var/run/docker.sock
export COMPOSE_FILE=docker-compose.yml
echo $DOCKER_HOST

# Windows (PowerShell)
$env:DOCKER_HOST="unix:///var/run/docker.sock"
$env:COMPOSE_FILE="docker-compose.yml"
echo $env:DOCKER_HOST
```

**Validation**: Environment variables are set and retrievable.

---

**Completion**: All exercises passed ✅
