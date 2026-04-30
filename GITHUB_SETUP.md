# Final Setup: GitHub Secrets & Trigger Build

## Step 1: Create Docker Hub Access Token

1. Go to https://hub.docker.com/settings/security
2. Click **New Access Token**
3. Name: `GitHub Actions`
4. Copy the token (you'll only see it once)

## Step 2: Add GitHub Secrets

1. Go to https://github.com/danielcyrus/eyened
2. Click **Settings** (top menu)
3. Left sidebar → **Secrets and variables** → **Actions**
4. Click **New repository secret**
5. Create two secrets:

   **Secret 1:**
   - Name: `DOCKER_HUB_USERNAME`
   - Value: `danielcyrus`
   
   **Secret 2:**
   - Name: `DOCKER_HUB_PASSWORD`
   - Value: (paste your access token from Step 1)

## Step 3: Trigger Build

The workflow is already committed. To trigger it:

**Option A: Push a new commit**
```bash
git add DOCKER_HUB_CLEANUP.md DOCKER_BUILD.md
git commit -m "Add Docker Hub setup documentation"
git push origin main
```

**Option B: Manually trigger in GitHub**
1. Go to https://github.com/danielcyrus/eyened/actions
2. Click **Build and Push Docker Images** workflow
3. Click **Run workflow** → **Run workflow**

## What Happens Next

The GitHub Actions workflow will:
1. Build `danielcyrus/eyened-client:latest` for `linux/amd64` + `linux/arm64`
2. Build `danielcyrus/eyened-server:latest` for `linux/amd64` + `linux/arm64`
3. Build `danielcyrus/eyened-worker:latest` for `linux/amd64` + `linux/arm64`
4. Push all images to Docker Hub
5. Tag with git SHA for version tracking

**Check progress:**
- https://github.com/danielcyrus/eyened/actions (workflow status)
- https://hub.docker.com/r/danielcyrus/eyened-client (Docker Hub repos after build completes)

Takes ~10-15 minutes per image build.

## Deployment

Once images are on Docker Hub, deploy with:
```bash
cd docker
export DOCKER_HUB_USERNAME=danielcyrus
docker compose up -d
```

All data stored at: `/mnt/qnap-rc/eyened-test`
