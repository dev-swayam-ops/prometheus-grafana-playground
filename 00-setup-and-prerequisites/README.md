# 00 - Setup and Prerequisites

## What You'll Learn
- System requirements for Prometheus and Grafana
- Installing Docker and Docker Compose
- Verifying installations
- Understanding monitoring architecture basics
- Preparing your environment for hands-on labs

## Prerequisites
- A Windows/Mac/Linux machine with internet access
- Administrator access to install software
- At least 4GB RAM and 10GB disk space
- Basic command-line knowledge

## Key Concepts

### System Requirements
- **CPU**: 2+ cores (minimum)
- **RAM**: 4GB minimum (8GB recommended)
- **Disk**: 20GB+ free space
- **Docker**: Version 20.10+
- **Docker Compose**: Version 1.29+

### Software Stack
- **Docker**: Container runtime for isolated environments
- **Docker Compose**: Orchestration tool for multi-container apps
- **Git**: Version control system
- **Text Editor**: VS Code, Vim, or similar

## Hands-on Lab: Environment Setup

### Step 1: Install Docker
**Windows/Mac**: Download Docker Desktop from [docker.com](https://docker.com)

**Linux (Ubuntu)**:
```bash
# Update package manager
sudo apt-get update

# Install Docker
sudo apt-get install docker.io -y

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### Step 2: Verify Docker Installation
```bash
docker --version
```

**Expected Output**:
```
Docker version 20.10.x, build xxxxx
```

### Step 3: Install Docker Compose
**Windows/Mac**: Included with Docker Desktop

**Linux**:
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Step 4: Verify Docker Compose
```bash
docker-compose --version
```

**Expected Output**:
```
Docker Compose version 1.29.x
```

### Step 5: Verify Docker Works
```bash
docker run hello-world
```

**Expected Output**:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Step 6: Clone Repository
```bash
git clone <your-repo-url>
cd prometheus-grafana-playground
```

### Step 7: Check File Structure
```bash
ls -la  # On Windows: dir
```

**Expected Output**:
```
00-setup-and-prerequisites/
01-prometheus-basics/
02-metrics-and-exporters/
...
README.md
```

## Validation

Run the following commands to verify setup:

```bash
# Check Docker is running
docker ps

# Check Compose
docker-compose --version

# Check Git
git --version

# Check directory structure
pwd  # On Windows: cd
```

All commands should return version information without errors.

## Cleanup

To remove Docker images and containers (optional):
```bash
docker system prune -a
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "docker: command not found" | Ensure Docker is installed and added to PATH |
| "Permission denied" on Linux | Add user to docker group: `sudo usermod -aG docker $USER` |
| "Cannot connect to Docker daemon" | Ensure Docker Desktop is running (Windows/Mac) |
| Insufficient disk space | Clean up images: `docker system prune` |
| Old Docker version | Update Docker to latest version |

## Troubleshooting

**Issue**: Docker daemon not running (Linux)
```bash
# Start Docker
sudo systemctl start docker

# Check status
sudo systemctl status docker
```

**Issue**: Cannot find docker-compose command
```bash
# Verify installation
which docker-compose

# Reinstall if needed
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

**Issue**: Out of disk space
```bash
# Check disk usage
df -h  # Linux/Mac
disk usage # Windows

# Clean Docker resources
docker system prune -a --volumes
```

## Next Steps

✅ Environment setup complete!

**Ready to proceed to**:
- [01-prometheus-basics](../01-prometheus-basics/README.md) - Learn Prometheus fundamentals
- [02-metrics-and-exporters](../02-metrics-and-exporters/README.md) - Understand metrics collection
