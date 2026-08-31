---
title: GitHub Actions — Implementation
category: ci-cd
topic: github-actions
tags: [learning, ci-cd, github-actions, implementation]
related: ["[[github-actions]]", "[[all-features]]"]
---

Hands-on work using a small Node.js repo on GitHub. Builds on the generic pipeline from [[../ci-cd-fundamentals/implementation|CI/CD Fundamentals' implementation]] — this file goes deeper into GitHub-Actions-specific features (composite actions, reusable workflows, concurrency, permissions). One fully worked example first, then a problem set — **problems 2+ are yours to build unaided**; only the expected final output is given.

## Prerequisites (for reference)

- A GitHub repository with a small Node.js project.
- Comfortable with the basic workflow shape from the fundamentals Topic (`on:`, `jobs:`, `steps:`).

---

## Worked example — a composite action for setup

**Goal:** extract the repeated "checkout + setup Node + install deps" steps into a reusable composite action, then use it from a workflow.

```yaml
# .github/actions/setup-project/action.yml
name: 'Setup Project'
description: 'Checkout, set up Node, and install dependencies'
runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
    - run: npm install
      shell: bash
```

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-project
      - run: npm test
```

**Why this works:** `actions/checkout@v4` must still run in the calling workflow (a composite action can't check out the repo on its own behalf implicitly the way `uses:` composition might suggest — it runs in the context of whatever job called it, but the repo needs to already be present on disk before `npm install` can succeed). The composite action bundles the Node setup and install into one referenced step (`uses: ./.github/actions/setup-project`), so any future workflow in this repo needing the same setup calls one line instead of duplicating three steps.

Running this on a push shows:

```
✅ CI / build-and-test
  ✓ Checkout
  ✓ Setup Project (setup-node + npm install)
  ✓ npm test
```

---

## Problem set

Build each of these yourself in your own GitHub repository. Only the expected final output is given — no solution YAML.

### Beginner

**1. `workflow_dispatch` with a typed input**
Add a second workflow triggered only by `workflow_dispatch`, with a required input `environment` (a string). Have the one step just `echo` the chosen environment.

Expected output:
```
Triggering the workflow manually from the GitHub Actions UI shows a form prompting for "environment" before the run starts.
The job log's echo step prints exactly the value you typed into that form.
```

**2. Restricting `GITHUB_TOKEN` permissions**
Add `permissions: contents: read` at the top level of your CI workflow. Confirm the workflow still runs and passes (since it only reads code and runs tests).

Expected output:
```
The workflow run completes successfully exactly as before — no functional change, since the job never needed write access.
```

### Intermediate

**3. Concurrency cancellation**
Add a `concurrency` block keyed on the branch ref with `cancel-in-progress: true`. Push two commits to the same branch within a few seconds of each other.

Expected output:
```
The Actions tab shows the first run transition to a "Cancelled" status shortly after the second run starts, and only the second (latest) run completes.
```

**4. Environment with required reviewer**
Configure a `production` environment in repo Settings requiring your own GitHub account as a reviewer. Add a `deploy` job (a placeholder `echo "deploying"` step) targeting `environment: production`, gated behind `needs: build-and-test`.

Expected output:
```
After build-and-test passes on a push to main, the deploy job shows "Waiting for review" in the Actions UI until you approve it, after which it runs and completes.
```

### Advanced

**5. Reusable workflow across two workflow files**
Create `.github/workflows/reusable-test.yml` using `on: workflow_call`, taking a `node-version` input, running install+test. Create a second workflow file that calls it twice via `uses:` with two different `node-version` values (e.g. 18 and 20), as separate jobs.

Expected output:
```
The Actions run shows two jobs, each internally running the reusable workflow's steps with a different Node version, both completing independently.
```

**6. Pinning an action to a commit SHA**
Take any `uses:` action reference in your workflow (e.g. `actions/checkout@v4`) and replace the version tag with the exact commit SHA that tag currently points to (findable on the action's GitHub repo "Tags" page). Re-run the workflow.

Expected output:
```
The workflow behaves identically to before (same effective version), but the YAML now pins an immutable commit SHA instead of a mutable tag — inspect the diff and be able to explain out loud why this is more supply-chain-safe.
```
