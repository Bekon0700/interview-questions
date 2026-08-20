---
title: GitHub Actions
category: ci-cd
topic: github-actions
tags: [learning, ci-cd, github-actions]
related: ["[[../ci-cd]]", "[[../ci-cd-fundamentals/ci-cd-fundamentals]]"]
---

> [!abstract] Overview
> **GitHub Actions** is GitHub's built-in CI/CD platform: pipelines ("workflows") are YAML files living in `.github/workflows/`, triggered by repository events (push, PR, schedule, manual dispatch), and run on GitHub-hosted or self-hosted **runners**. Its defining feature is the **Marketplace** of reusable, shareable **Actions** — pre-built steps anyone can plug into a workflow instead of writing raw shell commands.

## Backstory

GitHub Actions launched in 2018 (general availability in 2019), extending GitHub from "where the code lives" into "where the code is also built, tested, and deployed" — competing directly with standalone CI platforms like Travis CI, CircleCI, and self-hosted Jenkins. Its main strategic advantage is being **native to GitHub**: no separate account, no webhook wiring, no third-party service holding your repo credentials — workflows live in the same repo, trigger off the same events GitHub already tracks (PRs, issues, releases), and status checks show up directly on the PR.

## What it is

- A **workflow** is a YAML file (`.github/workflows/*.yml`) defining triggers (`on:`) and one or more **jobs**.
- Each **job** runs on a **runner** (a fresh VM per run, GitHub-hosted or self-hosted) and consists of a sequence of **steps** — either a shell command (`run:`) or a reusable **action** (`uses:`).
- **Actions** are the reusable building blocks — versioned, shareable units of workflow logic (e.g. `actions/checkout`, `actions/setup-node`) published to the **GitHub Marketplace**, referenced by `owner/repo@version`.
- Jobs within a workflow run in **parallel by default**; `needs:` creates explicit dependencies to force sequencing.
- **Secrets** and **environments** (with required reviewers) are configured in repository/organization settings, referenced in YAML via `${{ secrets.NAME }}`.

## Why it exists

Before GitHub Actions, using CI meant wiring a third-party service (Travis CI, CircleCI, Jenkins) to your GitHub repo via webhooks and OAuth, managing a separate config format, and often a separate billing relationship — extra setup friction and another system to keep in sync with repository events. GitHub Actions collapses that into the same platform already hosting the code, issues, and pull requests, and its Marketplace model means most common tasks (checkout, language setup, Docker build/push, deploy to a cloud provider) are a one-line `uses:` away instead of hand-written shell scripts.

## How it works (high level)

1. A repository event fires (push, PR opened, release published, schedule tick, manual `workflow_dispatch`).
2. GitHub matches the event against the `on:` triggers of every workflow file in `.github/workflows/`.
3. Matching workflows start; each `job` is scheduled onto a runner (GitHub provisions a fresh `ubuntu-latest`/`windows-latest`/`macos-latest` VM, or routes to a self-hosted runner with matching labels).
4. Steps run in order on that runner; a non-zero exit from any step fails the job (and, if other jobs `need` it, blocks them too).
5. Results post back as **status checks** on the commit/PR — combined with **branch protection rules**, a required check blocks merging until it passes.
6. Jobs targeting a protected **environment** (e.g. `production`) pause for manual approval from a designated reviewer before running their steps.
