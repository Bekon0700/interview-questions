---
title: CI/CD Fundamentals — All Features
category: ci-cd
topic: ci-cd-fundamentals
tags: [learning, ci-cd, features]
related: ["[[ci-cd-fundamentals]]"]
---

Comprehensive feature list for CI/CD concepts and tooling. Each feature includes why it was necessary and a concrete example.

## Pipeline-as-code

The pipeline definition (stages, triggers, steps) lives in a config file checked into the repository, not configured by hand in a UI.

> [!tip] Why this was necessary
> UI-configured pipelines aren't versioned, reviewable, or reproducible across environments — a change to the pipeline itself needs the same review/history as any other code change.

> [!example]
> A `.github/workflows/ci.yml` file defines the build/test pipeline; a PR that changes it goes through the same code review as an application change.

## Triggers

Rules for when a pipeline runs — on push, on pull request, on a schedule (cron), or manually.

> [!tip] Why this was necessary
> Different events warrant different pipelines: every push might run fast unit tests, while a nightly cron run might run a slow full regression suite that would be too slow to run on every commit.

> [!example]
> A pipeline runs unit tests on every push to any branch, but only runs the full end-to-end suite on pushes to `main` and on a nightly schedule.

## Build matrix

Running the same pipeline across multiple combinations of variables (OS, language version, dependency version) in parallel.

> [!tip] Why this was necessary
> A library that must support Node 18, 20, and 22 across Linux and macOS needs to verify all combinations, not just one — running them one after another instead of in parallel would slow feedback drastically.

> [!example]
> A GitHub Actions `strategy.matrix` runs the test suite across `node: [18, 20, 22]` × `os: [ubuntu-latest, macos-latest]` — 6 jobs in parallel instead of 6 sequential runs.

## Caching and incremental builds

Reusing dependency installs or build outputs from a previous run instead of redoing them from scratch every time.

> [!tip] Why this was necessary
> Reinstalling every dependency on every single pipeline run wastes minutes per run across potentially hundreds of runs a day — caching turns a 5-minute install into a 10-second cache restore.

> [!example]
> A pipeline caches `node_modules` (keyed by a hash of `package-lock.json`) so a run with no dependency changes skips `npm install` almost entirely.

## Branch protection / required checks

A repository setting that blocks merging a pull request until specified pipeline checks pass.

> [!tip] Why this was necessary
> A pipeline that runs but doesn't actually block anything is advisory only — teams need a hard gate so a red pipeline can't be merged and break `main` for everyone else.

> [!example]
> A GitHub branch protection rule on `main` requires the `build` and `test` checks to pass (and a review approval) before the "Merge" button becomes clickable.

## Environments and deployment gates

Named targets (e.g. `staging`, `production`) a pipeline can deploy to, often with required approvals or wait timers before deploying to sensitive ones.

> [!tip] Why this was necessary
> Not every environment should be treated the same — an automatic deploy to a staging sandbox is low-risk, but production deploys often need a human sign-off or a controlled rollout window.

> [!example]
> A GitHub Actions `environment: production` requires a designated reviewer to approve the deployment job before it proceeds, even though the `staging` environment deploys automatically.

## Secrets management

Encrypted storage for credentials (API keys, tokens, cloud credentials) that pipeline steps need, without putting them in the repository.

> [!tip] Why this was necessary
> Credentials committed to a git repo are effectively public forever (git history persists) — pipelines need a way to inject sensitive values at runtime without ever writing them to disk in the repo.

> [!example]
> A deploy step reads `secrets.AWS_ACCESS_KEY_ID` (configured in the platform's UI, encrypted at rest) as an environment variable, never checked into `ci.yml` itself.

## Rollback and deployment strategies (blue-green, canary)

Deployment techniques that reduce the blast radius of a bad release — running old and new versions side by side, or shifting traffic gradually.

> [!tip] Why this was necessary
> Deploying straight to 100% of production traffic means a bad release affects every user immediately; gradual or side-by-side rollout strategies let a bad deploy be caught and reverted before it impacts everyone.

> [!example]
> A canary deployment routes 5% of traffic to the new version for 10 minutes, watching error rates, before promoting it to 100% — or automatically rolling back if errors spike.

## Notifications and status reporting

Pipelines report pass/fail status back to the triggering commit/PR, and often to chat tools (Slack) or dashboards.

> [!tip] Why this was necessary
> A pipeline that runs silently in the background doesn't give developers the fast feedback CI is meant to provide — status needs to be visible where developers already are (the PR, their chat client).

> [!example]
> A failed pipeline run posts a red ❌ status check directly on the GitHub pull request and a message to a `#builds` Slack channel.
