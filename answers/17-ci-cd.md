---
title: CI/CD — Answers
topic: ci-cd
tags: [interview, ci-cd, devops, github-actions, docker]
related: ["[[14-docker-deployment]]", "[[06-system-design]]", "[[07-cv-deep-dive]]"]
---

# CI/CD — Answers

> [!abstract] How to use this note
> Each item embeds the **question**, a plain-English **explanation**, a worked **example**, and a **Pros / Cons** box. Numbers match [[questions/17-ci-cd|the questions file]]. This isn't explicitly on your CV — expect it as a natural follow-up to your Docker/deployment work, so lean on [[14-docker-deployment]] and [[07-cv-deep-dive]] for concrete tie-ins.

---

## Beginner

### 1. What is CI/CD?

> [!question] Q1
> What is CI/CD? What's the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?

**CI/CD** automates the path from "a developer pushed a commit" to "that change is verified and running in production," replacing manual build/test/deploy steps with a repeatable pipeline.

- **Continuous Integration (CI):** every change is automatically built and tested, frequently (ideally on every push), so integration bugs surface within minutes.
- **Continuous Delivery:** CI plus automatically producing a release-ready artifact/environment — a human still clicks "deploy" to production.
- **Continuous Deployment:** CI plus automatic deployment to production with no manual gate — every change that passes the pipeline ships.

> [!example]
> ```
> Push commit → CI builds + runs tests →
>   [Continuous Delivery] → artifact ready, human clicks "Deploy to prod"
>   [Continuous Deployment] → auto-deployed to prod, no click needed
> ```

> [!success] Pros / Cons
> **Pros:** fast feedback, fewer manual mistakes, smaller/safer releases.
> **Cons:** requires real test coverage to be safe, especially for Continuous Deployment — otherwise bugs just reach production faster.

> [!info] Further study
> - [Martin Fowler — Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
> - [Martin Fowler — Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html)

---

### 2. Pipeline as code

> [!question] Q2
> Why is "pipeline as code" preferred over configuring a pipeline by hand in a UI?

The pipeline definition (stages, triggers, steps) is a config file checked into the repo (e.g. `.github/workflows/ci.yml`, `Jenkinsfile`, `.gitlab-ci.yml`) instead of clicked together in a web UI. That means it's **versioned**, **reviewable** in a normal PR, and **reproducible** — anyone can see exactly what runs by reading the file, and a change to the pipeline goes through the same review process as an application change.

> [!example]
> ```yaml
> # .github/workflows/ci.yml — reviewable, diffable, rollback-able like any code change
> on: [push, pull_request]
> jobs:
>   test:
>     runs-on: ubuntu-latest
>     steps:
>       - uses: actions/checkout@v4
>       - run: npm test
> ```

> [!success] Pros / Cons
> **Pros:** code review for pipeline changes, git history/blame, easy to copy for a new repo.
> **Cons:** YAML for complex pipelines can get unwieldy; still needs the platform's UI for secrets and some settings.

---

### 3. Pipeline stages

> [!question] Q3
> What is a CI/CD pipeline stage/job, and what's a typical sequence of stages?

A **stage** (or job) is one logical step of the pipeline, running a set of commands, often on its own runner. A typical sequence: **build → lint → test → package (artifact/image) → deploy (staging → production)**. Later stages usually only run if earlier ones succeed.

> [!example]
> ```
> install → lint → unit tests → build → docker build/push → deploy staging → (manual approval) → deploy production
> ```

> [!success] Pros / Cons
> **Pros:** fails fast — a lint error stops the pipeline before wasting time on a full test run.
> **Cons:** too many sequential stages without parallelization slows total pipeline time.

---

### 4. CI runner

> [!question] Q4
> What is a build/CI runner (or agent)?

A **runner** (GitHub Actions) or **agent** (Jenkins) is the machine — often an ephemeral, containerized VM — that actually executes the pipeline's steps: checks out code, installs dependencies, runs commands. Hosted platforms (GitHub, GitLab, CircleCI) provide managed runners; teams can also run **self-hosted runners** for custom hardware, private network access, or cost control.

> [!example]
> ```yaml
> jobs:
>   build:
>     runs-on: ubuntu-latest   # GitHub-hosted runner, fresh VM per run
> ```

> [!success] Pros / Cons
> **Hosted pros:** zero maintenance, auto-scaled. **Hosted cons:** less control, can't reach private-network resources.
> **Self-hosted pros:** custom hardware/network access. **Self-hosted cons:** you own patching, scaling, and security.

---

### 5. Triggers

> [!question] Q5
> What triggers can start a pipeline (push, PR, schedule, manual)?

Common triggers: **push** (to any/specific branches), **pull_request** (often scoped to a target branch), **schedule** (cron, e.g. nightly full regression), and **manual/workflow_dispatch** (a human clicks "run"). Different triggers often run different subsets of the pipeline — e.g. fast tests on every push, a slow full suite only nightly.

> [!example]
> ```yaml
> on:
>   push:
>     branches: [main]
>   pull_request:
>     branches: [main]
>   schedule:
>     - cron: '0 2 * * *'   # nightly at 2am
>   workflow_dispatch: {}    # manual trigger button
> ```

---

### 6. Build artifact

> [!question] Q6
> What is a build artifact?

An **artifact** is the output a pipeline produces that later stages or a deploy step consume — a compiled binary, a bundled frontend, a `.jar`, or (most commonly for containerized apps) a **Docker image** pushed to a registry. Artifacts are usually tagged with a version or commit SHA so any past build can be redeployed exactly.

> [!example]
> ```bash
> docker build -t my-app:sha-abc123 .
> docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:sha-abc123
> # this exact image can be re-deployed later without rebuilding
> ```

---

## Intermediate

### 7. Basic workflow config

> [!question] Q7
> How do you configure a basic GitHub Actions / GitLab CI workflow to run tests on every push and pull request?

A minimal GitHub Actions workflow: check out the code, set up the runtime, install dependencies, run tests.

> [!example]
> ```yaml
> name: CI
> on:
>   push:
>     branches: [main]
>   pull_request:
>     branches: [main]
> jobs:
>   build-and-test:
>     runs-on: ubuntu-latest
>     steps:
>       - uses: actions/checkout@v4
>       - uses: actions/setup-node@v4
>         with: { node-version: '20' }
>       - run: npm install
>       - run: npm test
> ```

> [!success] Pros / Cons
> **Pros:** simple, no external infra needed, status shows directly on the PR.
> **Cons:** without branch protection this is purely advisory — a red pipeline doesn't stop anything by itself.

> [!tip] Interview tip
> Be ready to describe this exact YAML shape from memory — it's the most commonly asked "show me a pipeline" question.

---

### 8. Build matrix

> [!question] Q8
> What is a build matrix and why would you use one?

A **matrix** runs the same job across multiple combinations of variables (OS, language version, dependency version) **in parallel**, instead of sequentially. It's how you verify support across, say, 3 Node versions × 2 OSes without writing 6 separate job definitions.

> [!example]
> ```yaml
> strategy:
>   matrix:
>     node: [18, 20, 22]
>     os: [ubuntu-latest, macos-latest]
> runs-on: ${{ matrix.os }}
> steps:
>   - uses: actions/setup-node@v4
>     with: { node-version: ${{ matrix.node }} }
> ```

> [!success] Pros / Cons
> **Pros:** parallel execution keeps total time low despite testing many combinations.
> **Cons:** more runner minutes consumed (cost), and a matrix failure report can be noisier to triage.

---

### 9. Caching

> [!question] Q9
> How does dependency caching work in a CI pipeline, and what determines a cache hit vs miss?

A cache stores a directory (e.g. `node_modules` or npm's cache) between runs, keyed by a hash of something that changes only when dependencies change (typically `package-lock.json`). If the key matches a previous run's cache, it's restored and the install step can skip or shortcut most of the work; if the lockfile changed, the key changes and it's a cache miss, so a fresh install runs (and a new cache is saved for next time).

> [!example]
> ```yaml
> - uses: actions/cache@v4
>   with:
>     path: ~/.npm
>     key: npm-${{ hashFiles('package-lock.json') }}
> ```

> [!success] Pros / Cons
> **Pros:** turns multi-minute installs into seconds on unchanged dependencies.
> **Cons:** stale/incorrect cache keys can cause subtle bugs (using outdated deps silently) if the key doesn't actually capture what changed.

---

### 10. Secrets management

> [!question] Q10
> How do you manage secrets (API keys, cloud credentials) in a pipeline without committing them to the repo?

Secrets are stored encrypted in the CI platform itself (GitHub Actions Secrets, GitLab CI/CD variables) and injected as environment variables at runtime — never written into the YAML or the repo. Git history is effectively permanent, so anything committed (even briefly) should be treated as compromised and rotated.

> [!example]
> ```yaml
> - name: Deploy
>   run: ./deploy.sh
>   env:
>     AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
>     AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
> ```

> [!success] Pros / Cons
> **Pros:** encrypted at rest, masked in logs, scoped per repo/environment.
> **Cons:** still a real attack surface — a compromised third-party action with access to secrets can exfiltrate them (see Q19).

---

### 11. Required status checks

> [!question] Q11
> What is a required status check / branch protection rule, and why does it matter?

A **branch protection rule** on a branch (e.g. `main`) requires specified pipeline checks (and often a review approval) to pass before a pull request can be merged. Without this, a pipeline is purely informational — a developer could merge a red build anyway. This is the actual enforcement mechanism that turns "we have CI" into "broken code cannot reach main."

> [!example]
> ```
> GitHub → Settings → Branches → Branch protection rule for "main"
>   ✔ Require status checks to pass before merging
>       - build-and-test
> ```

---

### 12. Building and pushing Docker images in CI

> [!question] Q12
> How do you structure a pipeline to build a Docker image and push it to a registry (ECR, Docker Hub, GHCR)? (ties to your Docker work)

After tests pass, a separate job builds the Docker image (ideally the same multi-stage Dockerfile used for the 2.1GB → 170MB optimization — see [[14-docker-deployment]]), tags it (commit SHA and/or semver), authenticates to the registry, and pushes it. Deploy steps then reference that specific tag rather than `:latest`, so any past build stays reproducible and roll-back-able.

> [!example]
> ```yaml
> - name: Build and push
>   run: |
>     docker build -t $REGISTRY/my-app:${{ github.sha }} .
>     echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login $REGISTRY -u $REGISTRY_USER --password-stdin
>     docker push $REGISTRY/my-app:${{ github.sha }}
> ```

> [!success] Pros / Cons
> **Pros:** every commit produces a traceable, immutable, deployable image.
> **Cons:** registry storage/bandwidth costs; images need periodic cleanup/retention policy.

---

### 13. Monorepo / changed-only builds

> [!question] Q13
> What's the difference between a monorepo pipeline that rebuilds everything vs one that only builds what changed?

A naive monorepo pipeline reruns the full build/test suite for every package on every push, regardless of what changed — correct but wasteful as the repo grows. A change-aware pipeline uses path filters or dependency-graph tooling (e.g. Turborepo, Nx, or `git diff` on changed paths) to only build/test the packages actually affected by a given change (plus anything that depends on them), cutting pipeline time significantly on large repos.

> [!example]
> ```yaml
> - name: Detect changed packages
>   run: git diff --name-only origin/main... | grep '^packages/' | cut -d/ -f2 | sort -u
> # only run build/test for those packages
> ```

> [!success] Pros / Cons
> **Pros:** pipeline time scales with the size of a change, not the size of the whole repo.
> **Cons:** more complex to set up correctly; a badly configured dependency graph can silently skip a build that should have run.

---

## Advanced

### 14. Manual approval gate before production

> [!question] Q14
> How do you design a deployment pipeline with a manual approval gate before production?

Define an **environment** (e.g. GitHub Actions `environment: production`) with required reviewers configured. A deploy job targeting that environment pauses in a "waiting" state until one of the designated reviewers approves it in the UI — the job simply doesn't proceed until then. Staging can be a separate environment with no such requirement, so it deploys automatically.

> [!example]
> ```yaml
> deploy-prod:
>   needs: deploy-staging
>   environment: production   # requires manual approval, configured in repo settings
>   runs-on: ubuntu-latest
>   steps:
>     - run: ./deploy.sh production
> ```

> [!success] Pros / Cons
> **Pros:** keeps a human decision point for the highest-risk step without giving up automation everywhere else.
> **Cons:** adds latency to releases; if approvals become a rubber stamp, it's process theater rather than real safety.

---

### 15. Blue-green vs canary

> [!question] Q15
> Explain blue-green vs canary deployment strategies. How would you implement either in a CI/CD pipeline?

**Blue-green:** run two full environments ("blue" = current live, "green" = new version). Deploy the new version to green, verify it, then switch the router/load balancer to send all traffic to green. Rollback is instant — switch back to blue.

**Canary:** roll the new version out to a small percentage of traffic (e.g. 5%) first, watch error rates/latency, then gradually increase to 100% if healthy, or automatically roll back if metrics degrade.

> [!example]
> ```
> Blue-green: deploy v2 to "green" stack → smoke test → flip load balancer target from blue → green
> Canary: route 5% traffic to v2 → watch error rate for 10 min → 25% → 50% → 100% (or auto-rollback on error spike)
> ```

> [!success] Pros / Cons
> **Blue-green pros:** instant rollback, simple mental model. **Cons:** needs 2x infrastructure running at once.
> **Canary pros:** limits blast radius of a bad release automatically. **Cons:** more complex tooling (traffic shifting, automated health checks) to implement correctly.

---

### 16. Rollbacks

> [!question] Q16
> How do you handle rollbacks in an automated deployment pipeline?

Because every deployed artifact is an immutable, tagged image (Q6/Q12), rolling back is redeploying the previous known-good tag rather than reverting code and rebuilding. Many pipelines keep the last N deployed image tags available specifically for this. Automated rollback (in blue-green/canary setups) triggers this based on health-check/error-rate thresholds without waiting for a human.

> [!example]
> ```bash
> # No rebuild needed — just redeploy the previous tag
> kubectl set image deployment/my-app my-app=my-app:sha-<previous-good-commit>
> ```

> [!success] Pros / Cons
> **Pros:** fast recovery (minutes, not a full rebuild cycle); doesn't depend on git revert + re-run CI under pressure.
> **Cons:** requires disciplined artifact retention and that database/schema changes in the bad release are also backward-compatible with the rollback target.

---

### 17. End-to-end pipeline design

> [!question] Q17
> How would you design CI/CD for a Node/Next.js app with staging and production environments end-to-end (build → test → containerize → deploy)? (your CV: deployment story)

```
push/PR → lint + unit tests (fast feedback, blocks merge via branch protection)
        → on merge to main:
            build multi-stage Docker image (the 2.1GB → 170MB optimized build)
            push image tagged with commit SHA to registry
            deploy to staging automatically, run smoke tests
            deploy to production behind a manual approval gate (environment: production)
            production sits behind Nginx reverse proxy; deploy uses blue-green or rolling update for zero downtime
```

Tie this directly to your CV: the same Dockerfile that got the image from 2.1GB to 170MB is what CI builds and pushes on every merge; Nginx in front of the Node/Next.js app is what the pipeline's deploy step updates.

> [!tip] Interview tip
> Walk through this as a story end-to-end rather than listing features — interviewers want to see you reason about the whole flow, not recite pipeline YAML.

---

### 18. Keeping CI fast at scale

> [!question] Q18
> How do you keep CI pipelines fast as a codebase and test suite grow?

- Parallelize independent jobs (matrix builds, splitting test suites across shards).
- Cache dependencies and build outputs (Q9).
- Only build/test what changed in a monorepo (Q13).
- Move slow tests (full e2e, load tests) to a separate, less-frequent pipeline (nightly) instead of blocking every PR.
- Fail fast — run cheap checks (lint, type-check) before expensive ones (full test suite).

> [!success] Pros / Cons
> **Pros:** developers get feedback in the minutes range instead of tens of minutes, so they stay in flow and don't batch changes to avoid slow pipelines.
> **Cons:** aggressive parallelization/sharding and changed-only detection both add real configuration complexity and new ways for something to be silently skipped.

---

### 19. Third-party action security risks

> [!question] Q19
> What are the security risks of third-party actions/plugins in a pipeline, and how do you mitigate them?

A workflow step that runs a third-party action (`uses: some-org/some-action@v1`) executes arbitrary code with whatever permissions and secrets that job has access to. A compromised or malicious action can exfiltrate secrets, tamper with the build, or push unauthorized artifacts. Mitigations: pin actions to a full commit SHA (not a mutable tag like `@v1`), scope `GITHUB_TOKEN`/secrets permissions to the minimum a job needs, review third-party action source before adopting it, and prefer well-maintained/official actions.

> [!example]
> ```yaml
> # Risky: mutable tag, could change to malicious code without notice
> - uses: some-org/some-action@v1
> # Safer: pinned to an exact, auditable commit
> - uses: some-org/some-action@a1b2c3d4e5f6...
> ```

> [!success] Pros / Cons
> **Pros of pinning/scoping:** meaningfully reduces blast radius of a supply-chain compromise.
> **Cons:** pinned SHAs need manual bumping to get updates/security fixes — a small maintenance tax for the safety.

---

### 20. Database migrations in deployment

> [!question] Q20
> How do you handle database migrations safely as part of an automated deployment?

Run migrations as an explicit pipeline step, separate from and before the application restart — never implicitly on app boot in a way that could run concurrently from multiple instances. Prefer **backward-compatible, additive migrations** (add a nullable column, don't rename/drop in the same deploy) so the old app version keeps working against the new schema during a rolling deploy, then clean up destructive changes in a later deploy once the old version is fully retired.

> [!example]
> ```
> Deploy step 1: run migration (add new nullable column) — old and new app code both still work
> Deploy step 2: roll out new app version that writes to the new column
> Deploy step 3 (later, separate deploy): drop the old column once nothing reads it anymore
> ```

> [!success] Pros / Cons
> **Pros:** avoids the classic "migration ran but old pods are still running old code against the new schema" outage during a rolling deploy.
> **Cons:** requires discipline — splitting one "obvious" migration into multiple safe deploy steps feels slower than just doing it in one shot.

---

## Related notes

- [[14-docker-deployment]] — Dockerfile/image optimization that CI builds and pushes; Nginx and zero-downtime deployment strategies
- [[06-system-design]] — scaling and load balancing context for where CI/CD deploys land
- [[07-cv-deep-dive]] — tie pipeline answers back to your actual deployment story
- [[questions/17-ci-cd]] — Question list (companion to this answer note)

## References & Further Study

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Martin Fowler — Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
- [Martin Fowler — Continuous Delivery](https://martinfowler.com/bliki/ContinuousDelivery.html)
- [AWS — Blue/Green Deployments](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html)
- [GitHub — Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
