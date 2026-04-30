# Deploy Docker Images from GitHub on Target Machine

## Option 1: Docker Compose (Recommended - Pulls from Docker Hub)

On your target machine:

```bash
# Clone the repository
git clone https://github.com/danielcyrus/eyened.git
cd eyened/docker

# Set environment variables
export DOCKER_HUB_USERNAME=danielcyrus
export PORT=80

# Pull latest images and start services
docker compose up -d
```

This will:
- Pull `danielcyrus/eyened-client:latest`
- Pull `danielcyrus/eyened-server:latest`
- Pull `danielcyrus/eyened-worker:latest` (if needed)
- Start all services
- Mount storage at `/mnt/qnap-rc/eyened-test`

### Update to Latest

```bash
cd eyened/docker
docker compose pull
docker compose up -d
```

---

## Option 2: Manual Docker Run Commands

```bash
# Pull images
docker pull danielcyrus/eyened-client:latest
docker pull danielcyrus/eyened-server:latest
docker pull danielcyrus/eyened-worker:latest

# Run server
docker run -d \
  --name eyened-server \
  --env-file .env \
  -v /mnt/qnap-rc/eyened-test:/mnt/qnap-rc/eyened-test \
  -p 8000:8000 \
  danielcyrus/eyened-server:latest

# Run client
docker run -d \
  --name eyened-client \
  -p 4173:4173 \
  danielcyrus/eyened-client:latest

# Run nginx (file server + reverse proxy)
docker run -d \
  --name eyened-fileserver \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/templates/default.conf.template:ro \
  -v /mnt/qnap-rc/eyened-test/thumbnails:/storage/thumbnails:ro \
  --link eyened-server:server \
  --link eyened-client:client \
  nginx:latest
```

---

## Option 3: Build from Source on Target Machine

If you want to build locally instead of pulling pre-built images:

```bash
# Clone repository
git clone https://github.com/danielcyrus/eyened.git
cd eyened

# Build images locally
docker build -t danielcyrus/eyened-client:latest -f docker/Dockerfile.client .
docker build -t danielcyrus/eyened-server:latest -f docker/Dockerfile.server .
docker build -t danielcyrus/eyened-worker:latest -f docker/Dockerfile.worker .

# Start services
cd docker
docker compose up -d
```

⚠️ **Note:** This requires all build dependencies (Node.js, Python, PyTorch, CUDA) on target machine and takes much longer.

---

## Option 4: Use Docker Buildx for Multi-Arch Build

If target machine is arm64 (QNAP NAS) and you want to build locally:

```bash
# Create buildx builder
docker buildx create --name builder --use

# Build and load locally
docker buildx build \
  --load \
  --tag danielcyrus/eyened-client:latest \
  -f docker/Dockerfile.client \
  .

# Start services
cd docker
docker compose up -d
```

---

## Recommended Setup for QNAP NAS

For QNAP (arm64/v4):

```bash
# On QNAP target machine
ssh admin@your-qnap-ip

# Install Docker (if not present)
# QNAP has Container Station - use that or install Docker

# Clone repo
git clone https://github.com/danielcyrus/eyened.git
cd eyened/docker

# Set environment
export DOCKER_HUB_USERNAME=danielcyrus
export PORT=80

# Pull and start
docker compose --pull always up -d
```

The `--pull always` flag ensures you get the latest multi-arch image matching your system's architecture (arm64).

---

## File Storage

All data stored at: `/mnt/qnap-rc/eyened-test`

Make sure this directory exists on target machine:
```bash
sudo mkdir -p /mnt/qnap-rc/eyened-test
sudo chmod 755 /mnt/qnap-rc/eyened-test
```

---

## Verify Deployment

```bash
# Check running containers
docker ps

# View logs
docker logs eyened-server
docker logs eyened-client

# Test API
curl http://localhost:8000/health

# Test frontend
curl http://localhost:80
```

---

## Summary

| Method | Best For | Speed | Dependencies |
|--------|----------|-------|--------------|
| **Docker Compose** | Production, QNAP NAS | Fast (pull only) | Docker, docker-compose |
| **Manual docker run** | Simple deployments | Fast (pull only) | Docker |
| **Build locally** | Development, custom builds | Slow (build on target) | Full dev stack |
| **Buildx multi-arch** | Cross-platform builds | Medium | Docker, buildx |

**Start with Option 1 (Docker Compose)** - it's the easiest and fastest.
