# Deployment Guide

## Architecture Overview

Developer -> git push -> GitHub Actions -> ghcr.io -> AWS EC2 -> Live API

Pipeline stages: test -> build -> push -> deploy (via SSM)

### Components
- **GitHub Actions** — Automates testing, building, and deploying on every push to main
- **ghcr.io** — Stores Docker images tagged by commit SHA and latest
- **AWS SSM** — Sends deploy commands to EC2 securely without opening port 22
- **AWS EC2** — t3.micro Linux server running the Docker container

---

## EC2 Instance Setup

### Instance Details

| Item | Value |
|---|---|
| Instance type | t3.micro |
| OS | Amazon Linux 2023 |
| Region | us-east-1 |

### Security Group Rules

| Type | Port | Source | Purpose |
|---|---|---|---|
| HTTP | 80 | 0.0.0.0/0 | Public app access |
| SSH | 22 | MYIP/32 only | Personal admin access |

> SSH is restricted to a single IP address for security.
> The pipeline deploys via AWS SSM and does not require SSH access.

### Connect to EC2 from Windows

For manual administration/troubleshooting only:

```bash
ssh -i "deployready-key.pem" ec2-user@34.229.116.178
```

---

## Docker Installation on EC2

Commands run once after EC2 was created:

```bash
# Update all system packages
sudo dnf update -y

# Install Docker
sudo dnf install -y docker

# Start Docker service
sudo systemctl start docker

# Enable Docker to start automatically on reboot
sudo systemctl enable docker

# Add ec2-user to docker group to run without sudo
sudo usermod -aG docker ec2-user

# Apply group change in current session
newgrp docker

# Confirm Docker is installed
docker --version
```

---

## How the Pipeline Deploys

Every push to `main` triggers this sequence:

1. Test: Install dependencies and run `npm test`. If tests fail, the pipeline stops and nothing is deployed.
2. Build: Build the Docker image from `Dockerfile` and tag it with both commit SHA and `latest`.
3. Push: Push both tags to `ghcr.io`.
4. Deploy (via AWS SSM): Log in to `ghcr.io` on EC2, pull `latest`, replace the running container, and run a health check (`curl http://localhost/health`). If the check fails, roll back to the previous SHA image.

---

## Operations Guide

### Check if container is running
```bash
docker ps
```

### View live application logs
```bash
docker logs -f app
```

### Manually restart container
```bash
docker restart app
```

### Check health endpoint
```bash
curl http://34.229.116.178/health
```

### Stop container
```bash
docker stop app
```

---

## GitHub Secrets Setup

| Secret Name | Description | Where to Get It |
|---|---|---|
| `REGISTRY_TOKEN` | GitHub PAT to push/pull images | github.com/settings/tokens |
| `EC2_HOST` | EC2 public IP address | AWS Console → EC2 → Instances |
| `EC2_USER` | EC2 login username | Always `ec2-user` for Amazon Linux |
| `AWS_ACCESS_KEY_ID` | IAM user access key | AWS Console → IAM → Users |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key | AWS Console → IAM → Users |
| `AWS_REGION` | AWS region | `us-east-1` |
| `EC2_INSTANCE_ID` | EC2 instance ID | AWS Console → EC2 → Instances |

### How to Add Secrets

GitHub Repo → Settings → Secrets and variables
→ Actions → New repository secret

---

## Rollback

### Automatic Rollback

The pipeline automatically rolls back if the health check fails after deploy:

1. Start new container
2. Run `curl http://localhost/health`
3. If health check fails:
  - stop new container
  - redeploy previous SHA image
  - fail the pipeline

### Manual Rollback

```bash
docker stop app
docker rm app
docker run -d --name app \
  --restart unless-stopped \
  -p 80:3000 \
  -e PORT=3000 \
  ghcr.io/singizwa1/amalitech-deg-project-based-challenges:PREVIOUS_SHA
```

Replace `PREVIOUS_SHA` with any previous commit SHA from GitHub Actions history.

---

## Bonus Features

### Automatic Rollback on Health Check Failure

If the newly deployed container fails its health check, the pipeline
automatically redeploys the previous working image. This ensures
zero downtime from bad deployments.

### AWS SSM Instead of SSH

Deployment uses AWS Systems Manager instead of SSH. This means:

- Port 22 stays restricted to personal IP only
- No SSH keys needed in the pipeline
- More secure and professional deployment method