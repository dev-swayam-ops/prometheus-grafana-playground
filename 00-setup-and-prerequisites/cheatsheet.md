# Cheatsheet: Setup and Prerequisites

## Docker Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker --version` | Check Docker version | `docker --version` |
| `docker run` | Create and run container | `docker run -d nginx` |
| `docker ps` | List running containers | `docker ps -a` (include stopped) |
| `docker stop <id>` | Stop running container | `docker stop abc123` |
| `docker rm <id>` | Remove container | `docker rm abc123` |
| `docker images` | List local images | `docker images` |
| `docker pull <image>` | Download image from registry | `docker pull ubuntu` |
| `docker build -t <name> .` | Build image from Dockerfile | `docker build -t myapp .` |
| `docker logs <id>` | View container output | `docker logs -f abc123` |
| `docker exec -it <id> bash` | Execute command in container | `docker exec -it abc123 bash` |
| `docker inspect <id>` | Show detailed info | `docker inspect abc123` |

## Docker Compose Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker-compose --version` | Check Compose version | `docker-compose --version` |
| `docker-compose up` | Start services | `docker-compose up -d` |
| `docker-compose down` | Stop services | `docker-compose down` |
| `docker-compose ps` | List Compose services | `docker-compose ps` |
| `docker-compose logs` | View service logs | `docker-compose logs -f` |
| `docker-compose exec <service>` | Execute in service | `docker-compose exec app bash` |
| `docker-compose build` | Build images | `docker-compose build` |
| `docker-compose restart` | Restart services | `docker-compose restart` |

## Docker Network & Volume Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker network create <name>` | Create network | `docker network create monitoring` |
| `docker network ls` | List networks | `docker network ls` |
| `docker network rm <name>` | Remove network | `docker network rm monitoring` |
| `docker network inspect <name>` | Network details | `docker network inspect bridge` |
| `docker volume create <name>` | Create volume | `docker volume create data` |
| `docker volume ls` | List volumes | `docker volume ls` |
| `docker volume rm <name>` | Remove volume | `docker volume rm data` |
| `docker volume inspect <name>` | Volume details | `docker volume inspect data` |

## System & Cleanup Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker system prune` | Remove unused resources | `docker system prune -a` |
| `docker system df` | Show Docker disk usage | `docker system df` |
| `docker stats` | Show live resource usage | `docker stats` |
| `docker info` | System-wide info | `docker info` |

## Git Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `git --version` | Check Git version | `git --version` |
| `git config --global user.name` | Set Git username | `git config --global user.name "Name"` |
| `git config --global user.email` | Set Git email | `git config --global user.email "email@example.com"` |
| `git clone <url>` | Clone repository | `git clone https://github.com/repo.git` |
| `git status` | Show status | `git status` |
| `git add <file>` | Stage changes | `git add .` |
| `git commit -m "msg"` | Commit changes | `git commit -m "Initial setup"` |
| `git push` | Push to remote | `git push origin main` |
| `git pull` | Fetch and merge | `git pull origin main` |

## System Commands (Linux/Mac)

| Command | Purpose | Example |
|---------|---------|---------|
| `free -h` | Show RAM usage | `free -h` |
| `df -h` | Show disk usage | `df -h` |
| `du -sh <path>` | Show directory size | `du -sh .` |
| `ps aux` | List processes | `ps aux \| grep docker` |
| `top` | Monitor system resources | `top` |
| `uname -a` | Show system info | `uname -a` |

## System Commands (Windows PowerShell)

| Command | Purpose | Example |
|---------|---------|---------|
| `Get-Volume` | Show disk info | `Get-Volume` |
| `wmic logicaldisk get size,freespace` | Disk space | `wmic logicaldisk get size,freespace` |
| `Get-Process` | List processes | `Get-Process docker` |
| `tasklist` | List running tasks | `tasklist \| findstr docker` |
| `systeminfo` | System information | `systeminfo` |

## Quick Setup Reference

```bash
# 1. Install Docker & Compose (see README for full steps)
# 2. Verify installation
docker --version
docker-compose --version

# 3. Clone repository
git clone <repo-url>
cd prometheus-grafana-playground

# 4. Configure Git
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 5. Check system resources
free -h  # or appropriate command for your OS

# 6. Verify Docker daemon
docker ps

# 7. Start first service (when ready)
docker-compose up -d
```

---

**Quick Reference**: Keep this open while setting up your environment!
