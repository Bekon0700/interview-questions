---
title: Docker & Deployment — Answers
topic: docker-deployment
tags: [interview, fullstack, docker, deployment, nginx, kubernetes]
related: ["[[02-nextjs]]", "[[03-nodejs]]", "[[07-cv-deep-dive]]"]
---

# Docker & Deployment — Answers

> [!abstract] How to use this note
> Each item has the **question**, a plain-English **explanation**, an **example** (Dockerfile, Compose, or commands), and a **Pros / Cons** box. Numbers match [[questions/14-docker-deployment|the questions file]]. Questions **8, 9, and 15** tie directly to your CV Docker win (2.1GB → 170MB).

---

## Beginner

### 1. What is Docker and what problem does it solve?

> [!question] Q1
> What is Docker and what problem does it solve? What is the difference between an image and a container?

In plain terms, **Docker** packages your application together with everything it needs to run — code, runtime, libraries, and config — into a portable unit called a **container**. The classic problem it solves is *"works on my machine"* — when dev, staging, and production all run the same image, environment drift disappears.

An **image** is the immutable blueprint: a read-only stack of layers built from a Dockerfile. A **container** is a running instance of that image — it gets its own process space and a thin writable layer on top. You build an image once, then start one or many containers from it.

> [!example]
> ```bash
> # Build the blueprint
> docker build -t my-app:1.0 .

> # Run two independent instances from the same image
> docker run -d -p 3000:3000 --name app-a my-app:1.0
> docker run -d -p 3001:3000 --name app-b my-app:1.0
> ```

> [!success] Pros / Cons
> **Pros:** Consistent environments, fast startup, dense packing on one host, easy to version and roll back.  
> **Cons:** Shared-kernel isolation is weaker than a full VM; learning curve for networking, volumes, and production orchestration.

> [!info] Further study
> - [Docker overview — What is a container?](https://docs.docker.com/get-started/docker-overview/)

---

### 2. What is a Dockerfile?

> [!question] Q2
> What is a Dockerfile? Explain common instructions (FROM, WORKDIR, COPY, RUN, CMD, EXPOSE, ENV).

A **Dockerfile** is a text recipe that tells Docker how to build an image, step by step. Each instruction usually creates a new cached layer.

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image to start from (e.g. `node:20-alpine`) |
| `WORKDIR` | Sets the working directory inside the container |
| `COPY` | Copies files from your build context into the image |
| `RUN` | Executes a command at **build time** (install deps, compile) |
| `ENV` | Sets environment variables available at build and runtime |
| `EXPOSE` | Documents which port the app listens on (informational) |
| `CMD` | Default command when the container **starts** |

> [!example]
> ```dockerfile
> FROM node:20-alpine
> WORKDIR /app
> COPY package.json package-lock.json ./
> RUN npm ci --omit=dev
> COPY . .
> ENV NODE_ENV=production
> EXPOSE 3000
> CMD ["node", "server.js"]
> ```

> [!success] Pros / Cons
> **Pros:** Reproducible builds, version-controlled infrastructure, clear separation of build vs runtime steps.  
> **Cons:** Poor instruction ordering wastes cache; `RUN` layers bloat the image if you don't clean up in the same layer.

> [!info] Further study
> - [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

---

### 3. CMD vs ENTRYPOINT

> [!question] Q3
> What is the difference between `CMD` and `ENTRYPOINT`?

Both define what runs when a container starts, but they behave differently.

**`ENTRYPOINT`** sets the fixed executable — it always runs. **`CMD`** supplies default arguments (or a default command if no ENTRYPOINT is set). When both are present, `CMD` args are appended to `ENTRYPOINT`. You can override `CMD` at `docker run`; overriding `ENTRYPOINT` requires `--entrypoint`.

Use **ENTRYPOINT** for the main binary (e.g. `node server.js`) and **CMD** for default flags you might change per environment.

> [!example]
> ```dockerfile
> ENTRYPOINT ["node", "server.js"]
> CMD ["--port", "3000"]
> ```
> ```bash
> # Runs: node server.js --port 3000
> docker run my-app

> # Runs: node server.js --port 8080  (CMD overridden)
> docker run my-app --port 8080
> ```

> [!success] Pros / Cons
> **ENTRYPOINT pros:** Container always behaves like the intended app; good for wrapper scripts.  
> **ENTRYPOINT cons:** Less flexible for one-off debug commands without `--entrypoint`.  
> **CMD pros:** Easy to override at runtime.  
> **CMD cons:** Entire command is replaced when overridden — you lose the default executable.

> [!tip] Interview tip
> For production Node/Next.js images, prefer `CMD ["node", "server.js"]` in a minimal runner stage — simple and override-friendly.

---

### 4. Container registry

> [!question] Q4
> What is a container registry (Docker Hub, ECR)?

A **container registry** is a store for Docker images — like GitHub for binaries. You **push** built images after CI and **pull** them on servers or orchestrators at deploy time. Tags (e.g. `my-app:v1.2.3`, `my-app:sha-abc123`) identify versions.

Common registries: **Docker Hub** (public default), **AWS ECR**, **Google Artifact Registry**, **GitHub Container Registry (GHCR)**.

> [!example]
> ```bash
> docker build -t my-app:1.0.0 .
> docker tag my-app:1.0.0 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0
> docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:1.0.0
> ```

> [!success] Pros / Cons
> **Pros:** Central versioning, CI/CD integration, private repos for production images, rollback by re-deploying an old tag.  
> **Cons:** Registry costs and bandwidth; must scan images for vulnerabilities; tag discipline required (never rely on `:latest` alone in production).

> [!info] Further study
> - [Docker Hub](https://docs.docker.com/docker-hub/)
> - [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)

---

### 5. .dockerignore

> [!question] Q5
> What is `.dockerignore` and why is it important?

`.dockerignore` works like `.gitignore`: it excludes files from the **build context** sent to the Docker daemon. Without it, Docker uploads everything in the folder — including `node_modules`, `.git`, test fixtures, and local `.env` files — slowing builds and risking secrets in layers.

> [!example]
> ```dockerignore
> node_modules
> .git
> .next
> .env*
> *.md
> coverage
> __tests__
> ```

> [!success] Pros / Cons
> **Pros:** Faster builds, smaller context, better layer caching, reduced secret-leak risk.  
> **Cons:** Over-aggressive ignores can omit files the build needs — test your Dockerfile after changes.

> [!info] Further study
> - [Docker — .dockerignore](https://docs.docker.com/build/concepts/context/#dockerignore-files)

---

### 6. Container vs virtual machine

> [!question] Q6
> What is the difference between a container and a virtual machine?

A **virtual machine (VM)** virtualizes hardware and runs a full guest operating system on a hypervisor. Each VM is heavy (GBs of disk, minutes to boot) but has strong isolation.

A **container** shares the host OS kernel and isolates only processes, filesystem, and network namespaces. Containers start in milliseconds and use far less memory — you can run dozens on hardware that fits one VM.

> [!example]
> | | VM | Container |
> |---|---|---|
> | Isolation | Full OS + hypervisor | Kernel namespaces + cgroups |
> | Startup | Minutes | Milliseconds |
> | Size | GBs | MBs |
> | Use case | Mixed OS workloads, legacy apps | Microservices, CI, cloud-native apps |

> [!success] Pros / Cons
> **Container pros:** Fast, dense, portable, ideal for microservices and CI.  
> **Container cons:** Weaker isolation than VMs; all containers on a host share one kernel.  
> **VM pros:** Strong isolation, run different OS families on one host.  
> **VM cons:** Slow to start, expensive to operate at scale.

> [!info] Further study
> - [Docker — Containers vs VMs](https://docs.docker.com/get-started/docker-overview/#what-is-a-container)

---

## Intermediate

### 7. Image layers and caching

> [!question] Q7
> What are Docker image layers and how does layer caching work? How do you order instructions to maximize caching? (your CV)

Each Dockerfile instruction that modifies the filesystem creates an **immutable layer**. On rebuild, Docker reuses cached layers from the top down until the first instruction whose inputs changed — then it rebuilds that layer and **every layer after it**.

To maximize cache hits, order instructions from **least frequently changing** to **most frequently changing**. The classic pattern: copy `package.json` / lockfile, run `npm ci`, **then** copy source code. Code changes won't invalidate the expensive dependency-install layer.

> [!example]
> ```dockerfile
> # GOOD — deps cached unless package files change
> COPY package.json package-lock.json ./
> RUN npm ci
> COPY . .
> RUN npm run build

> # BAD — any source change re-runs npm ci
> COPY . .
> RUN npm ci && npm run build
> ```

> [!success] Pros / Cons
> **Pros:** Cached layers cut CI build time from minutes to seconds on typical code-only commits.  
> **Cons:** Stale cache can hide dependency updates if lockfiles aren't copied first; `--no-cache` rebuilds everything when debugging.

> [!tip] CV tip
> At Rokomari, separating the dependency layer from the build layer meant daily frontend commits didn't re-download hundreds of MB of `node_modules` — a practical win interviewers notice when you tie caching to CI speed, not just theory.

---

### 8. Multi-stage builds

> [!question] Q8
> What is a multi-stage build and why is it useful? (your CV: image size reduction)

A **multi-stage build** uses multiple `FROM` stages in one Dockerfile. An early **build stage** compiles the app with full dev tooling; a final **runner stage** copies only the production artifacts into a minimal base image. Stages you don't copy from are discarded — they never ship to production.

This was central to your **2.1GB → 170MB** reduction: the final image contains no TypeScript compiler, no devDependencies, no full source tree.

> [!example]
> ```dockerfile
> FROM node:20-alpine AS builder
> WORKDIR /app
> COPY . .
> RUN npm ci && npm run build

> FROM node:20-alpine AS runner
> WORKDIR /app
> COPY --from=builder /app/dist ./dist
> CMD ["node", "dist/main.js"]
> ```

> [!success] Pros / Cons
> **Pros:** Dramatically smaller images, smaller attack surface, no build tools in production.  
> **Cons:** More complex Dockerfile; must know exactly which artifacts the runner needs.

> [!info] Further study
> - [Docker — Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

---

### 9. Reducing Docker image size

> [!question] Q9
> How do you reduce Docker image size? List techniques. (your CV: 2.1GB → 170MB)

Image bloat usually comes from shipping build tools, dev dependencies, and unused files. Stack these techniques:

1. **Multi-stage builds** — only copy runtime artifacts to the final stage.
2. **Small base images** — `alpine`, `slim`, or `distroless` instead of full Debian.
3. **Production dependencies only** — `npm ci --omit=dev` in the runner (or rely on Next.js standalone tracing).
4. **`.dockerignore`** — exclude `node_modules`, `.git`, tests, docs.
5. **Combine RUN steps** — install and clean caches in one layer: `RUN npm ci && npm cache clean --force`.
6. **Next.js `output: 'standalone'`** — ships only traced runtime files, not all of `node_modules`.
7. **Non-root user** — security best practice (doesn't shrink size but belongs in production images).

> [!example]
> ```dockerfile
> # next.config.js — enable standalone output
> module.exports = { output: 'standalone' };
> ```
> ```bash
> # Verify size before/after
> docker images my-app
> docker history my-app:latest --human --no-trunc
> ```

> [!success] Pros / Cons
> **Pros:** ~12× size reduction (your CV), faster deploys/pulls, lower registry cost, quicker cold starts.  
> **Cons:** Each technique adds Dockerfile complexity; standalone output may miss edge-case static files you must copy manually.

> [!tip] Interview tip
> Lead with the **before/after number** (2.1GB → 170MB), then explain **why** (full node_modules + dev deps in one stage) and **how** (multi-stage + standalone + alpine + .dockerignore). See [[07-cv-deep-dive#20 Docker optimization 2.1GB → 170MB]].

---

### 10. Docker Compose

> [!question] Q10
> What is Docker Compose and when do you use it?

**Docker Compose** defines multi-container applications in a YAML file — services, networks, volumes, environment variables — and starts them with one command. It's ideal for **local development** and simple multi-service setups, not usually for large-scale production (orchestrators handle that).

> [!example]
> ```yaml
> # docker-compose.yml
> services:
>   app:
>     build: .
>     ports: ["3000:3000"]
>     environment:
>       DATABASE_URL: postgres://db:5432/myapp
>     depends_on:
>       db:
>         condition: service_healthy
>   db:
>     image: postgres:16-alpine
>     volumes: [pgdata:/var/lib/postgresql/data]
>     healthcheck:
>       test: ["CMD-SHELL", "pg_isready -U postgres"]
>       interval: 5s
> volumes:
>   pgdata:
> ```

> [!success] Pros / Cons
> **Pros:** One command spins up app + DB + Redis; reproducible dev environments; service DNS (`db`, `redis`) works out of the box.  
> **Cons:** Not a production orchestrator — no built-in rolling updates, autoscaling, or multi-host scheduling.

> [!info] Further study
> - [Docker Compose overview](https://docs.docker.com/compose/)

---

### 11. Configuration and secrets

> [!question] Q11
> How do you pass configuration/secrets into a container?

**Configuration** (non-sensitive): environment variables (`-e`, Compose `environment`, `env_file`), or mounted config files (read-only volumes).

**Secrets** (sensitive): inject at runtime from a secrets manager or orchestrator — never bake into the image or commit to Git. Options: Docker/Kubernetes secrets, AWS Secrets Manager, HashiCorp Vault, or CI-injected env vars at deploy time.

> [!example]
> ```bash
> # Runtime env (config)
> docker run -e NODE_ENV=production -e PORT=3000 my-app

> # Secret from file (Compose)
> # secrets:
> #   db_password:
> #     file: ./secrets/db_password.txt
> ```
> ```yaml
> # Kubernetes-style (conceptual)
> env:
>   - name: DATABASE_URL
>     valueFrom:
>       secretKeyRef:
>         name: app-secrets
>         key: database-url
> ```

> [!success] Pros / Cons
> **Pros:** Same image runs in dev/staging/prod with different config; secrets rotate without rebuilding.  
> **Cons:** Env vars can leak in logs/process listings; prefer secret mounts or managed secret stores in production.

> [!info] Further study
> - [Docker — Secrets](https://docs.docker.com/engine/swarm/secrets/)
> - [12-factor app — Config](https://12factor.net/config)

---

### 12. Volumes vs bind mounts

> [!question] Q12
> What are volumes and bind mounts? When use each?

Both persist data outside a container's ephemeral filesystem.

**Volumes** are Docker-managed storage in a host directory Docker controls. They survive container deletion, are portable across hosts (with plugins), and are the default for production data (DB files, uploads).

**Bind mounts** map a specific host path into the container. They're ideal for **local development** — mount your source code for live reload without rebuilding the image.

> [!example]
> ```bash
> # Volume — persistent DB data
> docker run -v pgdata:/var/lib/postgresql/data postgres:16

> # Bind mount — live code reload in dev
> docker run -v $(pwd):/app -v /app/node_modules my-app
> ```

> [!success] Pros / Cons
> **Volumes pros:** Docker-managed, backup-friendly, works well in production.  
> **Volumes cons:** Less transparent path on host.  
> **Bind mounts pros:** Direct host file access, perfect for dev hot-reload.  
> **Bind mounts cons:** Host path dependency, OS-specific paths, not ideal for production portability.

> [!info] Further study
> - [Docker — Volumes](https://docs.docker.com/storage/volumes/)
> - [Docker — Bind mounts](https://docs.docker.com/storage/bind-mounts/)

---

### 13. Container networking

> [!question] Q13
> How do you handle networking between containers?

Containers on the **same Docker network** resolve each other by **service name** (built-in DNS). Compose creates a default network automatically — your app connects to `postgres:5432` or `redis:6379` by hostname.

Expose services to the **host** with port mapping (`-p 3000:3000`). Use **separate networks** to isolate groups (e.g. frontend network vs internal DB network).

> [!example]
> ```yaml
> services:
>   api:
>     networks: [frontend, backend]
>   db:
>     networks: [backend]   # not reachable from host unless you publish ports
> networks:
>   frontend:
>   backend:
> ```
> ```bash
> # Manual network
> docker network create app-net
> docker run --network app-net --name api my-api
> docker run --network app-net --name db postgres:16
> ```

> [!success] Pros / Cons
> **Pros:** Service discovery without hard-coded IPs; network isolation improves security.  
> **Cons:** Debugging cross-container connectivity requires understanding bridge networks, DNS, and published vs internal ports.

> [!info] Further study
> - [Docker — Networking overview](https://docs.docker.com/network/)

---

### 14. Health checks

> [!question] Q14
> What is a health check and why does it matter?

A **health check** periodically verifies the container is actually working — not just running. Docker's `HEALTHCHECK` instruction (or orchestrator probes in Kubernetes) hits an endpoint like `/health` or runs a command. Failed checks trigger restarts; **readiness** probes control whether traffic is routed to the instance.

Health checks enable **reliable rollouts** — orchestrators wait until new containers are healthy before draining old ones.

> [!example]
> ```dockerfile
> HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
>   CMD wget -qO- http://localhost:3000/api/health || exit 1
> ```
> ```yaml
> # Kubernetes readiness probe (conceptual)
> readinessProbe:
>   httpGet:
>     path: /api/health
>     port: 3000
>   initialDelaySeconds: 5
>   periodSeconds: 10
> ```

> [!success] Pros / Cons
> **Pros:** Self-healing, safe rollouts, load balancers skip broken instances.  
> **Cons:** Poorly designed checks (too aggressive, wrong endpoint) cause flapping restarts; must distinguish liveness vs readiness.

> [!info] Further study
> - [Docker — HEALTHCHECK](https://docs.docker.com/reference/dockerfile/#healthcheck)
> - [Kubernetes — Configure liveness/readiness/startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

---

## Advanced

### 15. Next.js Dockerfile walkthrough (2.1GB → 170MB)

> [!question] Q15
> Walk through your Dockerfile for the Next.js image size optimization in detail. (your CV)

**The problem:** The original image was ~**2.1GB** because it copied the entire project — full `node_modules` (including devDependencies), build cache, source, and tooling — into a single production stage.

**The fix:** A three-stage pipeline plus Next.js **standalone output** so the runner ships only traced runtime files on Alpine (~**170MB**, roughly **12× smaller**).

**Stage 1 — `deps`:** Install dependencies in isolation. Cached unless `package.json` / lockfile change.  
**Stage 2 — `builder`:** Copy `node_modules` from deps, copy source, run `next build`. Standalone tracing produces `.next/standalone` with only required runtime files.  
**Stage 3 — `runner`:** Minimal Alpine image. Copy only `public`, `.next/standalone`, and `.next/static`. Run as non-root. No compiler, no devDeps, no source.

> [!example]
> **`next.config.js`**
> ```js
> /** @type {import('next').NextConfig} */
> const nextConfig = {
>   output: 'standalone',
> };
> module.exports = nextConfig;
> ```

> **`.dockerignore`**
> ```dockerignore
> node_modules
> .next
> .git
> .env*
> coverage
> __tests__
> *.md
> Dockerfile
> docker-compose*.yml
> ```

> **Full multi-stage Dockerfile**
> ```dockerfile
> # ── Stage 1: dependencies (cached unless lockfile changes) ──
> FROM node:20-alpine AS deps
> RUN apk add --no-cache libc6-compat
> WORKDIR /app
> COPY package.json package-lock.json ./
> RUN npm ci

> # ── Stage 2: build (standalone output) ──
> FROM node:20-alpine AS builder
> WORKDIR /app
> COPY --from=deps /app/node_modules ./node_modules
> COPY . .
> ENV NEXT_TELEMETRY_DISABLED=1
> RUN npm run build

> # ── Stage 3: production runner (~170MB) ──
> FROM node:20-alpine AS runner
> WORKDIR /app
> ENV NODE_ENV=production
> ENV NEXT_TELEMETRY_DISABLED=1

> RUN addgroup --system --gid 1001 nodejs \
>  && adduser --system --uid 1001 nextjs

> COPY --from=builder /app/public ./public
> COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
> COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

> USER nextjs
> EXPOSE 3000
> ENV PORT=3000
> ENV HOSTNAME="0.0.0.0"

> HEALTHCHECK --interval=30s --timeout=3s --start-period=15s --retries=3 \
>   CMD wget -qO- http://127.0.0.1:3000/api/health || exit 1

> CMD ["node", "server.js"]
> ```

> ```bash
> docker build -t rokomari-web:optimized .
> docker images rokomari-web:optimized
> # REPOSITORY        TAG         SIZE
> # rokomari-web      optimized   ~170MB  (was ~2.1GB)
> ```

> [!success] Pros / Cons
> **Pros:** ~12× smaller image → faster CI pushes, quicker pulls on deploy, lower ECR/storage cost, smaller attack surface, faster cold starts on ECS/K8s.  
> **Cons:** Standalone may miss untraced static assets (copy manually if needed); multi-stage Dockerfile is harder for juniors to maintain; Alpine + native modules sometimes need extra packages (`libc6-compat`).

> [!tip] CV interview script
> *"The old Dockerfile did a single-stage copy of everything — 2.1GB. I split it into deps → builder → runner, enabled `output: 'standalone'` so Next traces only the files the server imports, used Alpine, `.dockerignore`, and a non-root user. Final image ~170MB. That cut deploy time and registry cost on every release during the Next.js migration."*  
> Cross-reference: [[07-cv-deep-dive#20 Docker optimization 2.1GB → 170MB]], [[02-nextjs#23 Docker image 2.1GB → 170MB]].

> [!info] Further study
> - [Next.js — output: 'standalone'](https://nextjs.org/docs/app/api-reference/config/next-config-js/output)
> - [Next.js — With Docker example](https://github.com/vercel/next.js/tree/canary/examples/with-docker)

---

### 16. Reverse proxy (Nginx)

> [!question] Q16
> What is a reverse proxy (Nginx) and why put it in front of a Node app? (your CV: Nginx)

A **reverse proxy** sits in front of your app and handles incoming client traffic on its behalf. **Nginx** terminates TLS (HTTPS), serves static files, compresses responses (gzip/brotli), load-balances across multiple app instances, rate-limits abusive clients, and routes paths — e.g. `/api` → Node, `/` → static CDN.

Node focuses on application logic; Nginx handles **edge concerns** that Node handles less efficiently at scale.

> [!example]
> ```nginx
> upstream node_app {
>     server app1:3000;
>     server app2:3000;
> }
> server {
>     listen 443 ssl;
>     gzip on;
>     location /_next/static/ {
>         alias /var/www/static/;
>         expires 1y;
>     }
>     location / {
>         proxy_pass http://node_app;
>         proxy_set_header Host $host;
>         proxy_set_header X-Real-IP $remote_addr;
>     }
> }
> ```

> [!success] Pros / Cons
> **Pros:** TLS termination offloaded, efficient static serving, load balancing, caching, DDoS/rate-limit protection.  
> **Cons:** Another component to configure and monitor; misconfigured proxy headers can break apps or leak internal topology.

> [!info] Further study
> - [Nginx — Beginner's guide](https://nginx.org/en/docs/beginners_guide.html)
> - [Nginx — HTTP load balancing](https://nginx.org/en/docs/http/load_balancing.html)

---

### 17. Zero-downtime deployments

> [!question] Q17
> How do you achieve zero-downtime deployments (rolling, blue-green, canary)?

The goal: users never see errors while you ship a new version. All strategies rely on **health checks** and **graceful connection draining**.

**Rolling update:** Replace instances gradually — some old, some new always serve traffic. Default in Kubernetes/ECS.

**Blue-green:** Run the new version (green) alongside the old (blue). Switch the load balancer once green is healthy. Rollback = switch back instantly.

**Canary:** Route a small percentage of traffic to the new version, monitor error rate and latency, then ramp up or roll back.

> [!example]
> ```yaml
> # Kubernetes rolling update (conceptual)
> spec:
>   strategy:
>     type: RollingUpdate
>     rollingUpdate:
>       maxUnavailable: 0
>       maxSurge: 1
> ```

> [!success] Pros / Cons
> **Rolling pros:** Simple, built into orchestrators. **Cons:** Mixed versions run briefly — must handle backward-compatible DB/API changes.  
> **Blue-green pros:** Instant rollback, clean cutover. **Cons:** Double infrastructure cost during deploy.  
> **Canary pros:** Limits blast radius. **Cons:** Requires traffic splitting and good observability.

> [!info] Further study
> - [Kubernetes — Rolling updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
> - [AWS — Blue/green deployments](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html)

---

### 18. Deploy Node/Next.js to production

> [!question] Q18
> How would you deploy a Node/Next.js app to production (CI/CD pipeline, containers, orchestration)?

A typical production pipeline:

1. **Trigger** — push/merge to main.
2. **CI** — lint, unit tests, integration tests, build Docker image.
3. **Push** — tag and push image to registry (ECR/GHCR).
4. **Deploy** — update orchestrator/service (ECS, Kubernetes, or PaaS) with rolling/blue-green strategy.
5. **Migrate** — run DB migrations safely (often as a separate job before or during deploy).
6. **Verify** — health checks pass, smoke tests, monitor error rate for 15–30 minutes.

Manage env/secrets per environment. Never build different images per env — inject config at runtime.

> [!example]
> ```yaml
> # GitHub Actions (simplified)
> jobs:
>   deploy:
>     runs-on: ubuntu-latest
>     steps:
>       - uses: actions/checkout@v4
>       - run: docker build -t $REGISTRY/my-app:${{ github.sha }} .
>       - run: docker push $REGISTRY/my-app:${{ github.sha }}
>       - run: kubectl set image deployment/my-app app=$REGISTRY/my-app:${{ github.sha }}
> ```

> [!success] Pros / Cons
> **Pros:** Repeatable, auditable deploys; easy rollback to a previous image tag.  
> **Cons:** Pipeline complexity; must coordinate migrations, secrets, and backward-compatible releases.

> [!tip] Interview tip
> Mention your stack concretely: GitHub Actions → ECR → ECS/K8s, with health checks and rollback by redeploying the previous tag.

---

### 19. Orchestration (Kubernetes)

> [!question] Q19
> What is orchestration (Kubernetes) at a high level? What problems does it solve?

**Container orchestration** automates running containers across many machines. **Kubernetes (K8s)** is the dominant orchestrator — it schedules pods, restarts failed containers, load-balances services, rolls out updates, scales replicas, and manages config/secrets cluster-wide.

Problems it solves that Docker Compose can't: multi-host scheduling, self-healing, autoscaling, zero-downtime rollouts, service discovery at scale, and declarative desired state ("run 5 replicas of this image").

> [!example]
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: nextjs-app
> spec:
>   replicas: 3
>   selector:
>     matchLabels:
>       app: nextjs
>   template:
>     spec:
>       containers:
>         - name: app
>           image: my-registry/nextjs:1.0.0
>           ports: [{ containerPort: 3000 }]
>           livenessProbe:
>             httpGet: { path: /api/health, port: 3000 }
> ```

> [!success] Pros / Cons
> **Pros:** Production-grade reliability, autoscaling, ecosystem (Helm, operators), cloud-native standard.  
> **Cons:** Steep learning curve, operational overhead, overkill for small teams or single-service apps.

> [!info] Further study
> - [Kubernetes — Concepts](https://kubernetes.io/docs/concepts/)
> - [Kubernetes — Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---

### 20. Graceful shutdown

> [!question] Q20
> How do you handle graceful shutdown of a containerized Node service? (also in Node file)

When a container stops or scales down, the platform sends **SIGTERM**. Your Node process should:

1. Stop accepting new connections.
2. Finish in-flight requests within a timeout.
3. Close DB, Redis, and queue connections cleanly.
4. Exit with code 0.

In Kubernetes, combine **readiness probe** removal + **`preStop` hook** (sleep a few seconds) so the load balancer drains traffic before the pod is killed. Docker's default stop grace period is 10 seconds — tune with `stop_grace_period` if needed.

> [!example]
> ```js
> const server = app.listen(PORT);

> process.on('SIGTERM', () => {
>   server.close(async () => {
>     await db.end();
>     await redis.quit();
>     process.exit(0);
>   });
>   setTimeout(() => process.exit(1), 10000); // force exit after 10s
> });
> ```

> [!success] Pros / Cons
> **Pros:** No dropped requests during deploys; clean connection pool release.  
> **Cons:** Long-running requests may be cut off if timeout is too short; must handle SIGTERM in every long-lived worker, not just HTTP servers.

> [!tip] See also
> Full Node.js treatment: [[03-nodejs]] graceful shutdown section.

> [!info] Further study
> - [Kubernetes — Pod lifecycle — termination](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)
> - [Node.js — process signal events](https://nodejs.org/api/process.html#signal-events)

---

### 21. Monitoring and logging

> [!question] Q21
> How do you monitor and log a production containerized application?

**Logging:** Emit **structured JSON logs** to **stdout/stderr** — the platform (CloudWatch, Loki, ELK) collects them. Never log secrets. Include request IDs for correlation.

**Metrics:** Export CPU, memory, request rate, latency (p50/p95/p99), and error rate to Prometheus/Grafana or a hosted APM (Datadog, New Relic).

**Tracing:** OpenTelemetry or APM traces follow requests across services (API → DB → Redis).

**Alerting:** Set alerts on SLOs — error rate spikes, latency degradation, pod restarts, disk pressure.

> [!example]
> ```js
> logger.info({ reqId, method, path, status, durationMs }, 'request completed');
> ```
> ```yaml
> # Prometheus scrape annotation (K8s)
> metadata:
>   annotations:
>     prometheus.io/scrape: "true"
>     prometheus.io/port: "9090"
> ```

> [!success] Pros / Cons
> **Pros:** Fast incident diagnosis, capacity planning, proof that deploys are healthy.  
> **Cons:** Observability stack costs money and ops time; log volume can explode without sampling/retention policies.

> [!info] Further study
> - [OpenTelemetry — Documentation](https://opentelemetry.io/docs/)
> - [Prometheus — Getting started](https://prometheus.io/docs/prometheus/latest/getting_started/)
> - [Grafana Loki](https://grafana.com/docs/loki/latest/)

---

### 22. Horizontal scaling and load

> [!question] Q22
> How do you scale a containerized app horizontally and manage load?

**Horizontal scaling** means running **more container instances** behind a load balancer instead of giving one container more CPU/RAM. Requirements:

1. **Stateless app tier** — session/state in Redis or DB, not in-memory on one instance.
2. **Load balancer** — distributes requests round-robin or by least connections.
3. **Health checks** — traffic only hits healthy instances.
4. **Autoscaling** — scale on CPU, request rate, or queue depth (HPA in K8s, ECS Service Auto Scaling).
5. **Caching and queues** — absorb spikes (CDN, Redis, background workers).

> [!example]
> ```yaml
> # Kubernetes Horizontal Pod Autoscaler (conceptual)
> apiVersion: autoscaling/v2
> kind: HorizontalPodAutoscaler
> spec:
>   minReplicas: 2
>   maxReplicas: 20
>   metrics:
>     - type: Resource
>       resource:
>         name: cpu
>         target:
>           type: Utilization
>           averageUtilization: 70
> ```

> [!success] Pros / Cons
> **Pros:** Handles traffic spikes, fault tolerance (one instance dying doesn't take down the app), linear capacity growth.  
> **Cons:** Requires stateless design and shared stores; autoscaling lag can miss sudden spikes without pre-warming or queue buffering.

> [!info] Further study
> - [Kubernetes — Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
> - [AWS ECS — Service Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)

---

## Related notes

- [[02-nextjs]] — `output: 'standalone'`, migration context, Docker size optimization
- [[03-nodejs]] — Graceful shutdown, production Node patterns
- [[07-cv-deep-dive]] — STAR answers for 2.1GB → 170MB and deployment wins
- [[06-system-design]] — Scaling, load balancing, and zero-downtime in system design
- [[questions/14-docker-deployment]] — Question list (companion to this answer note)

---

## References & Further Study

### Docker
- [Docker Documentation](https://docs.docker.com/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Docker — Best practices](https://docs.docker.com/build/building/best-practices/)

### Next.js & deployment
- [Next.js — output: 'standalone'](https://nextjs.org/docs/app/api-reference/config/next-config-js/output)
- [Next.js — With Docker example](https://github.com/vercel/next.js/tree/canary/examples/with-docker)
- [Vercel — Production checklist](https://nextjs.org/docs/app/building-your-application/deploying/production-checklist)

### Reverse proxy & edge
- [Nginx — Documentation](https://nginx.org/en/docs/)
- [Nginx — HTTP load balancing](https://nginx.org/en/docs/http/load_balancing.html)

### Orchestration & cloud
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes — Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [AWS ECS — Developer guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/)
- [AWS — Blue/green deployments](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html)

### Observability
- [OpenTelemetry](https://opentelemetry.io/docs/)
- [Prometheus](https://prometheus.io/docs/introduction/overview/)
- [Grafana — Loki](https://grafana.com/docs/loki/latest/)
