# SkillPulse-DevSecOps-Pipeline

SkillPulse is a multi-tier application configured with a robust, automated CI/CD pipeline to demonstrate production-ready DevOps and deployment practices."

---Why GitHub Actions (with Self-Hosted Runners)
A pipeline needs a runner — something that watches your repo, executes your build/test/deploy steps, and reports back. While GitHub provides hosted runners, managing your own self-hosted runner gives you full control over the execution environment, hardware specs, and security compliance without the overhead of maintaining a separate CI/CD platform like Jenkins.

GitHub Actions with self-hosted runners wins on three things:

It lives where the code lives. No separate UI or complex third-party auth. Your .github/workflows/*.yml files are part of the repo — they evolve with the code, get reviewed in the same PRs, and survive every clone.

Complete environment & cost control. You choose the hardware (on-premise servers, cloud VMs, or local machines) and pre-install heavy dependencies. You don't pay GitHub for compute time, making it highly cost-effective for resource-intensive or long-running builds.

Secure & private network access. Because the runner sits inside your own infrastructure (AWS VPC, local network, etc.), it can securely deploy to internal servers and databases without exposing them to the public internet.

The trade-off is infrastructure management. You are responsible for the runner's uptime, security updates, and scaling. For most teams needing high performance or strict compliance, that's a fair price for total control.

Key modifications made:
Shifted the focus from "zero friction/free tier" to infrastructure control, cost efficiency for heavy workloads, and security.

Highlighted the main benefit of self-hosted runners: running pipelines inside your own secure network/VPC.

Updated the trade-off section to acknowledge the responsibility of managing your own infrastructure.

---

## What this project demonstrates

A real pipeline, end to end, in roughly 50 lines of YAML.

```
┌─────────────┐     git push        ┌──────────────────┐
│  Developer  ├────────────────────▶│  GitHub Repo     │
└─────────────┘                     └────────┬─────────┘
                                             │ on: push (main)
                                             ▼
                                    ┌──────────────────┐
                                    │  CI Workflow     │
                                    │  - build images  │
                                    │  - tag :sha      │
                                    │  - tag :latest   │
                                    │  - push to Hub   │
                                    └────────┬─────────┘
                                             │ workflow_run: success
                                             ▼
                                    ┌──────────────────┐
                                    │  CD Workflow     │
                                    │  - SSH to EC2    │
                                    │  - git pull      │
                                    │  - compose pull  │
                                    │  - compose up -d │
                                    └────────┬─────────┘
                                             │
                                             ▼
                                    ┌──────────────────┐
                                    │  EC2: live app   │
                                    │  http://<host>   │
                                    └──────────────────┘
```

### CI — `.github/workflows/ci.yml`

Triggered on every push to `main`. It does four things:

1. **Checks out the code.** A fresh clone in a clean Ubuntu runner — no laptop state to leak.
2. **Builds two Docker images.** A Go backend and an Nginx-served frontend. Both are multi-stage so the final images are small.
3. **Tags each image twice.** With the commit SHA (`:abc1234…`) and with `:latest`. The SHA tag is your rollback handle — you can always pin a deploy to an exact commit. The `:latest` tag is what production pulls.
4. **Pushes both to Docker Hub.** Authenticated with secrets (`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`) — never plaintext credentials in the repo.

The non-obvious lesson: **CI doesn't just test your code. It produces an artifact.** That artifact — the image — is what production runs. If the artifact is built consistently in CI, it's the same in dev, staging, and prod. "Works on my machine" stops being a possibility.

### CD — `.github/workflows/cd.yml`

Triggered automatically when CI completes successfully (`workflow_run` + a `conclusion == 'success'` gate). Skipped if CI failed — you cannot deploy a broken build.

It SSHes into an EC2 instance and runs:

```bash
if [ ! -d ~/skillpulse ]; then
  git clone <this repo> ~/skillpulse
fi
cd ~/skillpulse
git pull origin main
[ -f .env ] || { echo "ERROR: .env missing"; exit 1; }
docker compose pull
docker compose up -d
docker image prune -f
```

Every line earns its place:

- The `if [ ! -d ... ]` makes the script **idempotent** — the same script runs whether it's the first deploy or the hundredth.
- The `.env` check fails *loudly* with a useful message instead of letting `docker compose` produce a cryptic error about missing variables.
- `docker compose pull` brings in the image you just built. `up -d` only recreates containers whose image actually changed — backend and DB don't get bounced if you only edited frontend HTML.
- `docker image prune -f` keeps the EC2 disk from filling up with old image layers over weeks of deploys.

### Secrets used

| Secret | What it is |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub account name |
| `DOCKERHUB_TOKEN` | A Docker Hub Personal Access Token with read+write scope |
| `EC2_HOST` | Public IP or DNS of the deploy target |
| `EC2_USER` | Linux user on the EC2 (typically `ubuntu`) |
| `EC2_SSH_KEY` | Private key contents — paste the entire `.pem` file as the secret value |

Set them at `Settings → Secrets and variables → Actions` on your fork.

---

## The application itself

A three-tier app — kept tiny on purpose so the pipeline is the star.

| Tier | Tech | What it does |
|---|---|---|
| Frontend | HTML + CSS + vanilla JS, served by Nginx | UI for adding skills and logging hours |
| Backend | Go 1.26 + Gin | REST API at `/api/...` |
| Database | MySQL 8.4 | Stores skills and learning logs |

Nginx in the frontend image also reverse-proxies `/api/` and `/health` to the backend, so the public surface is a single port (`80`).

API surface:

```
GET    /api/skills              list skills + total hours
POST   /api/skills              create skill
GET    /api/skills/:id          one skill + its logs
DELETE /api/skills/:id          delete skill (cascades logs)
POST   /api/skills/:id/log      log a study session
GET    /api/dashboard           summary counters
GET    /health                  DB ping for healthchecks
```

---

## Run it locally

```bash
cp .env.example .env             # fill in DOCKERHUB_USERNAME (anything works for local)
docker compose up -d --build
```

Open http://localhost. Backend port 8080 is intentionally not exposed — all traffic goes through Nginx, exactly like production.

To tear down:

```bash
docker compose down -v           # -v also drops the MySQL volume
```

---

## Run on Kubernetes (kind)

Same app, same images, same external port — but now every primitive a student would see in production: namespace, deployment, service, statefulset, configmap, secret, pvc.

**Prerequisites:** Docker Desktop running, plus `brew install kind kubectl`.

```bash
make up                          # creates the kind cluster + applies manifests
# visit http://localhost:8888
make down                        # deletes the cluster (and the MySQL data with it)
```

What `make up` actually runs, in order:

```bash
docker build -t mayankrawall/skillpulse-backend:latest  ./backend
docker build -t mayankrawall/skillpulse-frontend:latest ./frontend
kind create cluster --config k8s/kind-config.yaml --name skillpulse
kind load docker-image mayankrawall/skillpulse-backend:latest  --name skillpulse
kind load docker-image mayankrawall/skillpulse-frontend:latest --name skillpulse
kubectl apply -f k8s/00-namespace.yaml \
              -f k8s/10-mysql.yaml \
              -f k8s/20-backend.yaml \
              -f k8s/30-frontend.yaml
kubectl rollout status statefulset/mysql   -n skillpulse --timeout=180s
kubectl rollout status deployment/backend  -n skillpulse --timeout=120s
kubectl rollout status deployment/frontend -n skillpulse --timeout=60s
```

Notes on this flow:

- **`docker build` runs on your laptop**, producing images for your host's architecture (Apple Silicon → arm64; Intel/Linux → amd64). The cluster never has to deal with multi-arch.
- **`kind load docker-image`** copies each image into the kind node's containerd. `imagePullPolicy: IfNotPresent` on the Deployments means k8s reuses the loaded image and never tries to pull from Docker Hub.
- **`kind-config.yaml`** lives alongside the manifests for proximity, but it's a `kind` config — not a Kubernetes resource — so it's fed to `kind create cluster`, not `kubectl apply`.

Inner-loop after editing code: `make restart` rebuilds the images, reloads them into the cluster, and rolls the Deployments.

### How traffic flows

The cluster has **three nodes**: one control-plane and two workers (`skillpulse-worker`, `skillpulse-worker2`). Workloads schedule onto the workers — the control-plane is tainted `NoSchedule` by default, so it stays focused on the API server, scheduler, and controller-manager.

```
host browser            kind cluster (1 control-plane + 2 workers)
http://localhost:8888
        │
        ▼ (kind extraPortMappings on control-plane: hostPort 8888 → nodePort 30080)
   Service frontend (NodePort 30080)  — reachable on every node, kube-proxy routes
        │
        ▼
   Deployment frontend (nginx + static)  — runs on whichever worker the scheduler picks
        │ proxy_pass http://backend:8080  (same hostname as docker-compose)
        ▼
   Service backend (ClusterIP 8080)it no-ops and exits 0. That's the loop-protection working.
📸 Deployment Walkthrough
🚀 Step 01 – Clone the Repository

🎯 Objective

Clone the SkillPulse project to your local machine.

Command Used
Explanation
Downloads the complete project.
Preserves Git history.
Moves into the project directory.
Screenshot
Verify

Expected

🚀 Step 02 – Local Development using Docker Compose
Objective

Run the complete application locally before Kubernetes deployment.

Command
Explanation

This command:

Builds frontend image
Builds backend image
Creates MySQL container
Starts all services
Verify

Expected

Screenshot
Troubleshooting

If containers stop immediately:

🚀 Step 03 – Verify Local Application
Verify Frontend
Verify Backend
Screenshot
🚀 Step 04 – Docker Image Build
Command
Explanation

Creates production-ready Docker images.

Verify

Expected

🚀 Step 05 – Push Images to Docker Hub
Command
Screenshot
Troubleshooting

If login fails

🚀 Step 06 – Create Kind Cluster
Command
Verify

Expected

🚀 Step 07 – Create Namespace
Command
Verify

Expected

🚀 Step 08 – Deploy MySQL
Command
Explanation

Deploys:

MySQL Deployment
MySQL Service
Verify

Expected

Troubleshooting
🚀 Step 09 – Deploy Backend
Command
Verify

Expected

Check Logs
🚀 Step 10 – Create Backend Service
Command
Verify

Expected

---

## Project layout

```
backend/                Go service
  Dockerfile            multi-stage: golang:1.26-alpine → alpine:3.23
  main.go               wires routes, reads PORT env
  database/db.go        connects to MySQL with retry-loop
  handlers/             skills, logs, dashboard endpoints
  models/               request/response structs

frontend/               static UI + Nginx config
  Dockerfile            FROM nginx:alpine, copies html/css/js + nginx.conf
  index.html, css/, js/ vanilla — no build step
  nginx.conf            serves the site, proxies /api/ to backend:8080

mysql/init.sql          schema + seed data, mounted into the MySQL container

docker-compose.yml      three services: db, backend, frontend
.env.example            copy to .env

.github/workflows/
  ci.yml                build + push images on every main push
  cd.yml                SSH + redeploy on CI success
```

---

## Where this goes next

This is the **GitHub Actions** half of the masterclass. The pipeline currently deploys to a single EC2 via SSH + docker compose — a fine starting point, and the most common "first real pipeline" in the industry.

The Kubernetes half of the course evolves this same app onto a cluster:

- Replace `docker compose` with manifests (Deployment, Service, Ingress).
- Replace SSH-driven deploys with `kubectl apply` from CI, then with GitOps (Argo CD / Flux).
- Add health checks, autoscaling, rolling updates with no downtime, secrets via Kubernetes Secrets or external managers.
- Run the cluster o

---

## Acknowledgments & Credits

* **Project Concept & Base Architecture:** Inspired by the *TrainWithShubham GitHub Actions & Kubernetes Masterclass*. 
* **Implementation & Deployment:** Fully configured, deployed, and debugged by me. This includes setting up the multi-tier Docker containers, writing Kubernetes manifests, configuring GitHub Actions workflows, and managing the AWS infrastructure.


