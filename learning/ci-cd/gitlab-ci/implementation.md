---
title: GitLab CI — Implementation
category: ci-cd
topic: gitlab-ci
tags: [learning, ci-cd, gitlab-ci, implementation]
related: ["[[gitlab-ci]]", "[[all-features]]"]
---

Hands-on work using GitLab.com (free tier includes shared runners — no local install needed) against a small Node.js repo. One fully worked example first, then a problem set — **problems 2+ are yours to build unaided**; only the expected final output is given.

## Prerequisites (for reference)

- A GitLab.com repository (project) with a small Node.js project (`npm test` runs at least one passing test).
- `.gitlab-ci.yml` at the repo root.

---

## Worked example — a basic build-and-test pipeline

**Goal:** run install and test automatically on every push, split into stages.

```yaml
# .gitlab-ci.yml
stages:
  - install
  - test

install-deps:
  stage: install
  image: node:20
  script:
    - npm install
  artifacts:
    paths:
      - node_modules/

run-tests:
  stage: test
  image: node:20
  script:
    - npm test
```

**Why this works:** `stages:` declares the order (`install` runs fully before `test` starts). Each job specifies `image:` (the Docker image the job runs in — GitLab CI runs jobs in containers by default on shared runners), and `script:` is the list of shell commands executed inside that container. Because each job runs in its own fresh container, `node_modules/` installed in `install-deps` wouldn't exist in `run-tests` without explicitly passing it via `artifacts:` — that's what makes it available to the next stage's job.

Pushing this file and any subsequent commit shows on the project's **CI/CD → Pipelines** page:

```
Pipeline #142 passed
  ✓ install-deps (install)
  ✓ run-tests (test)
```

---

## Problem set

Build each of these yourself in your own GitLab.com repository. Only the expected final output is given — no solution YAML.

### Beginner

**1. Only run on merge requests and main**
Modify the pipeline so jobs only run for merge request pipelines and pushes to `main`, using `rules:` — not on every arbitrary branch push.

Expected output:
```
Pushing directly to a random feature branch (with no open merge request) does NOT trigger a pipeline.
Opening a merge request from that branch DOES trigger a pipeline.
Pushing directly to main also triggers a pipeline.
```

**2. Add a lint stage**
Add a `lint` stage between `install` and `test` running `npm run lint`. Introduce a deliberate lint error and push it.

Expected output:
```
The pipeline view shows the lint job failing (red) in its own stage, and the test stage's job never starts.
```

### Intermediate

**3. Dependency caching**
Replace the artifact-passing approach for `node_modules` with `cache:` keyed on `package-lock.json`, so a second pipeline run with no dependency changes restores from cache instead of reinstalling.

Expected output:
```
First pipeline run: install job log shows a full npm install with no cache restore message.
Second run (no lockfile changes): install job log shows a cache restore step, and completes noticeably faster.
```

**4. `needs:` to skip stage waiting**
Add a second, unrelated slow job to the `test` stage (e.g. `sleep 60` to simulate a slow suite) alongside your existing fast test job. Add a `deploy` stage job that only `needs:` the fast test job, not the slow one.

Expected output:
```
The pipeline graph shows the deploy job starting as soon as the fast test job finishes, without waiting for the slow 60-second job in the same stage to complete.
```

### Advanced

**5. Review app per merge request**
Add a `review` job (stage `deploy`, triggered only on merge request pipelines via `rules:`) using `environment: name: review/$CI_COMMIT_REF_SLUG` that just echoes a fake "deployed" URL, plus an `on_stop` job to tear it down.

Expected output:
```
Opening a merge request shows a "View app" / environment link in the MR's pipeline widget pointing at the review environment name.
Closing or merging the MR triggers the stop job automatically, and the environment is marked "stopped" under Deployments → Environments.
```

**6. Protected variable for a fake production secret**
Add a CI/CD variable (Settings → CI/CD → Variables) named `PROD_TOKEN`, marked **Protected**. Mark `main` as a protected branch. Add a job on `main` only that echoes whether `PROD_TOKEN` is set, and try running the same job logic on an unprotected feature branch.

Expected output:
```
On main (protected branch): the job log shows PROD_TOKEN has a value (masked, but present).
On an unprotected feature branch: the job log shows PROD_TOKEN as empty/unset, confirming protected variables aren't exposed there.
```
