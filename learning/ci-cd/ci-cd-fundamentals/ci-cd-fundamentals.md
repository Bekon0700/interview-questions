---
title: CI/CD Fundamentals
category: ci-cd
topic: ci-cd-fundamentals
tags: [learning, ci-cd, fundamentals, pipelines]
related: ["[[../ci-cd]]"]
---

> [!abstract] Overview
> **CI/CD** is the automated pipeline that takes a code change from "committed" to "verified" to "released," replacing manual build/test/deploy steps with a repeatable, versioned process. **Continuous Integration (CI)** automatically builds and tests every change; **Continuous Delivery** extends that to producing a release-ready artifact with a manual deploy gate; **Continuous Deployment** removes that gate and ships straight to production.

## Backstory

CI/CD grew out of Extreme Programming's Continuous Integration practice (Kent Beck, late 1990s): merge and test every change frequently instead of letting branches diverge for weeks. Early tooling like CruiseControl (2001) automated "build + run tests on every commit." As teams pushed further — wanting not just tested code but *deployable* code on every commit — Continuous Delivery and Continuous Deployment emerged as natural extensions, popularized heavily through the 2010s alongside cloud infrastructure, containers, and hosted platforms like Jenkins, Travis CI, CircleCI, GitHub Actions, and GitLab CI.

## What it is

- A **pipeline**: a defined sequence of automated stages (e.g. build → lint → test → package → deploy) that runs on a trigger, most commonly a git push or pull request.
- Configuration lives as **code**, versioned alongside the project (e.g. `.github/workflows/*.yml`, `Jenkinsfile`, `.gitlab-ci.yml`), not as manual clicks in a UI.
- A **runner/agent** — a machine (often ephemeral/containerized) that actually executes the pipeline steps.
- **Artifacts** — the built output (binary, container image, package) produced by a pipeline run, which later stages or a deploy step consume.

## Why it exists

- Manually rebuilding and retesting after every change doesn't scale past a handful of contributors — it's slow and people skip it under deadline pressure.
- Integration problems (two branches touching the same code differently) are cheap to fix within minutes of being introduced, and expensive to fix weeks later once more code has been built on top of the broken assumption.
- Manual deployment steps (SSH in, copy files, restart a service, hope nothing was forgotten) are a major source of production incidents — a scripted pipeline does the same steps identically every time.
- Fast, automated feedback lets teams ship small changes constantly instead of large, risky "release day" batches.

## How it works (high level)

1. A developer pushes a commit or opens a pull request.
2. A trigger fires the pipeline; a runner checks out the code.
3. **CI stage**: dependencies install, the project builds, and automated tests (unit, integration, sometimes end-to-end) run. A failure here blocks the change — it's reported back on the commit/PR and nothing proceeds.
4. **Package/artifact stage**: on success, a deployable artifact is produced (e.g. a Docker image) and pushed to a registry.
5. **Delivery/deployment stage**: the artifact is deployed to one or more environments (staging, then production). Under Continuous Delivery this step requires a manual approval click; under Continuous Deployment it happens automatically once earlier stages pass.
6. Results (pass/fail, logs, deployed version) are reported back to the team, often blocking merges on a red pipeline via **branch protection rules**.
