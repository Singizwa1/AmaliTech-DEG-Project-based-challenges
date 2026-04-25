# DeployReady

![CI/CD Pipeline](https://github.com/Singizwa1/AmaliTech-DEG-Project-based-challenges/actions/workflows/deploy.yml/badge.svg)

A fully automated DevOps pipeline for a Node.js API — containerised with Docker,
tested and deployed automatically via GitHub Actions, running on AWS EC2.

---

## Architecture

1. Developer pushes code to `main`
2. GitHub Actions runs the pipeline:
   - Run tests (`npm test`)
   - Build Docker image
   - Push image to `ghcr.io`
   - Deploy to EC2 via AWS SSM
3. Live API is available at:
   - `http://34.229.116.178/health`

---

## Tech Stack

| Technology | Purpose |
|---|---|
| Node.js 20 | Application runtime |
| Docker | Containerisation |
| GitHub Actions | CI/CD pipeline |
| AWS EC2 (t3.micro) | Cloud server |
| AWS SSM | Secure deployment (no SSH needed) |
| ghcr.io | Docker image registry |

---

## API Endpoints

| Method | Route | Description |
|---|---|---|
| GET | `/health` | Returns `{"status":"ok"}` |
| GET | `/metrics` | Returns uptime and memory usage |
| POST | `/data` | Accepts JSON body and echoes it back |

---

## Local Development

### Prerequisites
- Docker Desktop installed
- Node.js 20 installed

### Run Locally
```bash
# Clone the repo
git clone https://github.com/Singizwa1/AmaliTech-DEG-Project-based-challenges.git

# Go to project directory
cd AmaliTech-DEG-Project-based-challenges/dev-ops/DeployReady

# Copy environment file
cp .env.example .env

# Start with Docker Compose
docker compose up --build

# Test the API
curl http://localhost:3000/health
```

---

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `PORT` | Port the app listens on | `3000` |
| `NODE_ENV` | Environment name | `production` |

---

## CI/CD Pipeline

Every push to `main` automatically:

1. **Tests** — runs `npm test` (stops if fails)
2. **Builds** — creates Docker image tagged with commit SHA
3. **Pushes** — uploads image to ghcr.io
4. **Deploys** — sends commands to EC2 via AWS SSM

See [Deployment.md](./Deployment.md) for full setup details.

---

## Decisions Made

| Decision | Reason |
|---|---|
| `node:20-alpine` | Smallest possible image size |
| Non-root user | Security best practice |
| t3.micro over t2.micro | Current free-tier option with better performance |
| ghcr.io registry | Free, integrated with GitHub |
| AWS SSM over SSH | Port 22 stays closed, more secure |
| Automatic rollback | Prevents bad deploys reaching users |