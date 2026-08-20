---
title: CI/CD — Category
topic: ci-cd
tags: [learning, category, ci-cd]
related: []
---

> [!abstract] What this Category is
> **CI/CD** (Continuous Integration / Continuous Delivery or Deployment) is the practice and tooling for automatically building, testing, and shipping code changes — turning "a developer pushed a commit" into "that change is verified and running in production" without manual, error-prone hand-offs.

## History — why this category was necessary

Before CI/CD, teams integrated code infrequently — branches lived for weeks or months, and merging them ("integration hell") surfaced conflicts and bugs late, when they were expensive to fix. Extreme Programming (late 1990s) introduced **Continuous Integration**: merge and test every change frequently, ideally multiple times a day, so problems surface within minutes instead of months. Tools like CruiseControl (2001) automated this.

As teams also wanted to ship faster and more reliably, **Continuous Delivery** extended the idea past testing into packaging and staging environments — every change that passes CI is automatically proven *releasable*, even if a human still clicks "deploy." **Continuous Deployment** goes one step further and removes that manual gate entirely: every change that passes the pipeline goes to production automatically. The rise of cloud infrastructure, containers, and hosted pipeline platforms (Jenkins, Travis CI, GitHub Actions, GitLab CI, CircleCI) made this practical for teams of any size, not just large orgs with dedicated release engineers.

## What problem this category solves

- **Catching integration bugs early** — automated build + test on every push means a broken change is caught in minutes, not discovered weeks later during a painful merge.
- **Removing manual, error-prone release steps** — hand-run deploy checklists are slow and inconsistent; a pipeline runs the same steps the same way every time.
- **Fast feedback for developers** — a red pipeline tells you immediately that a change broke something, before it reaches reviewers or production.
- **Enabling frequent, low-risk releases** — small, continuously-verified changes are far safer to ship than large, infrequent batches.
- **Reproducible builds and deployments** — pipelines codify the build/deploy process as versioned config, not tribal knowledge in someone's head.

## Alternative names / adjacent terms

- **Continuous Integration (CI)** — specifically the "build + test automatically on every change" part.
- **Continuous Delivery** — CI plus automatically producing a release-ready artifact/environment, with a manual gate before production.
- **Continuous Deployment** — CI plus fully automatic release to production, no manual gate.
- **Pipeline / build pipeline** — the concrete sequence of automated stages (build → test → deploy) that implements CI/CD for a project.
- **DevOps** — the broader cultural/organizational movement CI/CD tooling grew out of and supports.

## Topics in this Category

- [[ci-cd-fundamentals/ci-cd-fundamentals|CI/CD Fundamentals]] — the core concepts, pipeline stages, and practices shared across any CI/CD tool.

See [[comparison]] and [[recall]] once this Category has 2+ Topics (e.g. a specific platform like GitHub Actions or Jenkins).
