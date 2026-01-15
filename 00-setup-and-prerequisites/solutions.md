# Solutions: Setup and Prerequisites

## Solution 1: Docker Installation Verification
**Commands**:
```bash
docker --version
docker run hello-world
```

**Explanation**: 
- `docker --version` shows installed Docker version
- `docker run hello-world` confirms Docker daemon is running and functional

**Expected Output**:
```
Docker version 20.10.x, build xxxxx
Hello from Docker!
```

---

## Solution 2: Docker Compose Setup
**Command**:
```bash
docker-compose --version
```

**Explanation**: 
Verifies Docker Compose installation and checks version compatibility.

**Expected Output**:
```
Docker Compose version 1.29.x or higher
```

---

## Solution 3: Clone Repository
**Commands**:
```bash
git clone <your-repo-url>
cd prometheus-grafana-playground
ls -la
```

**Explanation**: 
- `git clone` downloads repository from remote
- `cd` navigates to the project directory
- `ls -la` lists all files/folders including hidden ones

**Expected Output**:
```
00-setup-and-prerequisites/
01-prometheus-basics/
02-metrics-and-exporters/
...
README.md
```

---

## Solution 4: Docker Daemon Status
**Commands**:
```bash
docker ps
docker info | grep "Containers"
```

**Explanation**: 
- `docker ps` lists running containers (empty if none running)
- `docker info` shows system-wide Docker info; grepping "Containers" shows count

**Expected Output**:
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS      NAMES
(empty if no containers)

Running: 0
Paused: 0
Stopped: 0
```

---

## Solution 5: System Resource Check
**Linux/Mac Commands**:
```bash
free -h
df -h
```

**Windows PowerShell Commands**:
```powershell
# RAM check
[math]::Round((Get-WmiObject -Class Win32_OperatingSystem).TotalVisibleMemorySize / 1MB, 2)

# Disk check
Get-Volume | Where-Object {$_.DriveType -eq 'Fixed'} | Select-Object DriveLetter, Size, SizeRemaining
```

**Explanation**: 
- Shows available RAM and disk space in human-readable format
- Verifies system meets minimum requirements

**Expected Output** (RAM ≥ 4GB, Disk ≥ 20GB):
```
RAM: 16Gi available
Disk: 100Gi total, 50Gi free
```

---

## Solution 6: Docker Network Test
**Commands**:
```bash
docker network create test-network
docker run --rm --name container1 --network test-network alpine ping -c 2 container2
```

**Explanation**: 
- `docker network create` establishes isolated network
- `docker run` with `--network` joins container to that network
- Containers can communicate via hostnames

**Expected Output**:
```
PING container2 (172.x.x.x): 56 data bytes
64 bytes from 172.x.x.x: seq=0 ttl=64 time=0.1ms
```

---

## Solution 7: Docker Volume Practice
**Commands**:
```bash
docker volume create test-volume
docker run --rm -v test-volume:/data alpine sh -c "echo 'data' > /data/test.txt"
docker volume inspect test-volume
```

**Explanation**: 
- `docker volume create` allocates persistent storage
- `-v` flag mounts volume to container path `/data`
- `docker volume inspect` shows volume details

**Expected Output**:
```json
[
  {
    "Name": "test-volume",
    "Driver": "local",
    "Mountpoint": "/var/lib/docker/volumes/test-volume/_data"
  }
]
```

---

## Solution 8: Git Configuration
**Commands**:
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global --list
```

**Explanation**: 
- `--global` flag sets config for all repositories on system
- Git requires name and email for commits
- `--list` displays all configuration settings

**Expected Output**:
```
user.name=Your Name
user.email=your.email@example.com
core.editor=vim
...
```

---

## Solution 9: Docker Cleanup
**Commands**:
```bash
docker system prune -a --volumes --force
docker images
docker ps -a
df -h
```

**Explanation**: 
- `docker system prune -a` removes all unused images, containers, volumes
- `--force` skips confirmation prompts
- Subsequent commands verify cleanup success

**Expected Output**:
```
Deleted Images: 5
Deleted Containers: 3
Deleted Volumes: 2

Total reclaimed space: 2.5GB
```

---

## Solution 10: Environment Variable Export
**Linux/Mac Commands**:
```bash
export DOCKER_HOST=unix:///var/run/docker.sock
export COMPOSE_FILE=docker-compose.yml
echo $DOCKER_HOST
```

**Windows PowerShell Commands**:
```powershell
$env:DOCKER_HOST = "unix:///var/run/docker.sock"
$env:COMPOSE_FILE = "docker-compose.yml"
echo $env:DOCKER_HOST
```

**Explanation**: 
- `export` (Linux) or `$env:` (Windows) sets environment variables
- Docker uses these variables for configuration
- `echo` verifies variable is set

**Expected Output**:
```
unix:///var/run/docker.sock
docker-compose.yml
```

---

**All Solutions Complete ✅**
