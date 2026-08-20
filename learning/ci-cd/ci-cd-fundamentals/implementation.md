---
title: CI/CD Fundamentals — Implementation
category: ci-cd
topic: ci-cd-fundamentals
tags: [learning, ci-cd, implementation]
related: ["[[ci-cd-fundamentals]]", "[[all-features]]"]
---

Hands-on work using **GitHub Actions** against a small Node.js repo (any small project with a `package.json` and a test script works — GitHub Actions requires no local install, only a GitHub repo). One fully worked example first, then a problem set — **problems 2+ are yours to build unaided**; only the expected final output is given.

## Prerequisites (for reference)

- A GitHub repository with a small Node.js project (`npm test` runs at least one passing test, e.g. via `node --test` or Jest).
- Workflow files live at `.github/workflows/*.yml`.

---

## Worked example — a basic build-and-test workflow

**Goal:** run `npm install` and `npm test` automatically on every push and pull request, and see the pass/fail status reported on GitHub.

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
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

**Why this works:** `on.push`/`on.pull_request` restricted to `main` means the workflow triggers whenever a commit lands on `main` or a PR targets it. `runs-on: ubuntu-latest` provisions a fresh, ephemeral virtual machine as the runner. `actions/checkout@v4` pulls the repo's code onto that runner (without it, there's no code to build). `actions/setup-node@v4` installs the requested Node version. The final two `run` steps are just shell commands executed on the runner, in order — if `npm test` exits non-zero, the job (and the whole workflow) is marked failed, which shows as a red ❌ on the commit/PR.

Pushing this file and any subsequent commit produces a check on GitHub that looks like:

```
✅ CI / build-and-test — all steps passed
```

or, if a test fails:

```
❌ CI / build-and-test — "Run tests" step failed
```

---

## Problem set

Build each of these yourself in your own GitHub repository. Only the expected final output is given — no solution YAML.

### Beginner

**1. Trigger on all branches, not just `main`**
Modify the workflow so it runs on pushes to *any* branch, not just `main`, while pull requests still only trigger for PRs targeting `main`.

Expected output:
```
Pushing a commit to a branch named e.g. "feature/x" triggers the workflow.
Opening a PR from "feature/x" into "main" also triggers the workflow.
Opening a PR into a branch other than "main" (if you create one) does NOT trigger it.
```

**2. Add a lint step**
Add an `npm run lint` step (add a trivial lint script/config if your project doesn't have one, e.g. via ESLint) that runs after install but before tests. Introduce one deliberate lint error and push it.

Expected output:
```
The workflow run shows the "Run tests" step never executes (or shows as skipped/not-run) because the lint step failed first and the job stops.
The check on GitHub shows ❌ with the failing step clearly identified as the lint step.
```

### Intermediate

**3. Build matrix across Node versions**
Modify the workflow to run the install+test steps across Node 18, 20, and 22 in parallel using a `strategy.matrix`.

Expected output:
```
The GitHub Actions run page shows 3 separate jobs (e.g. "build-and-test (18.x)", "build-and-test (20.x)", "build-and-test (22.x)") running concurrently, each with its own pass/fail status.
```

**4. Dependency caching**
Add caching for `node_modules` (or npm's cache directory) keyed on a hash of `package-lock.json`, so a second run with no dependency changes skips the full install.

Expected output:
```
First workflow run: cache step reports a "cache miss" and install takes its normal time.
Second run (no lockfile changes): cache step reports a "cache hit", and the install step completes noticeably faster than the first run.
```

### Advanced

**5. Required status check + branch protection**
Enable branch protection on `main` requiring the `build-and-test` job to pass before merging. Open a PR with a failing test.

Expected output:
```
The PR's "Merge" button is disabled/greyed out with a message referencing the required check, until the failing test is fixed and the workflow passes.
```

**6. Deploy stage with a manual approval gate**
Add a second job (e.g. `deploy`) that only runs after `build-and-test` succeeds and only on pushes to `main`, targeting a GitHub Actions `environment: production` that requires a manual reviewer approval before the job proceeds. The "deploy" step itself can just be a placeholder (e.g. `echo "deploying..."`).

Expected output:
```
After a push to main with passing tests, the workflow run shows the "deploy" job paused in a "Waiting for approval" state.
Approving it in the GitHub UI lets the job proceed and complete; rejecting it marks the job as cancelled/failed without running the deploy step.
```
