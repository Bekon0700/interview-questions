---
title: GitLab CI
category: ci-cd
topic: gitlab-ci
tags: [learning, ci-cd, gitlab-ci]
related: ["[[../ci-cd]]", "[[../ci-cd-fundamentals/ci-cd-fundamentals]]"]
---

> [!abstract] Overview
> **GitLab CI/CD** is GitLab's built-in pipeline system — conceptually similar to GitHub Actions (native to the hosting platform, config-as-code, hosted or self-hosted runners) but with its own YAML shape (`.gitlab-ci.yml`), its own execution model (**stages** running in sequence, **jobs** within a stage running in parallel by default), and a stronger historical emphasis on being a genuinely complete "single application for the whole DevOps lifecycle" (source control, CI/CD, container registry, security scanning) rather than a pipeline bolted onto a separately-scoped hosting platform.

## Backstory

GitLab CI launched in 2012, integrated into GitLab CE from 2015 onward, years before GitHub Actions (2018). GitLab's stated philosophy has long been "one application" spanning the entire DevOps lifecycle — planning, source control, CI/CD, container registry, security scanning, and monitoring — rather than a hosting platform with CI added on. This shows up concretely in GitLab CI's design: it's tightly integrated with GitLab's built-in Container Registry, its Auto DevOps feature (pipeline templates generated automatically for common project shapes), and native security/compliance scanning stages available without third-party actions.

## What it is

- A pipeline is defined in a single `.gitlab-ci.yml` file (or split across included files) at the repo root.
- Pipelines are organized into **stages** (e.g. `build`, `test`, `deploy`) that run **in order**; **jobs** within the same stage run **in parallel** by default.
- Work runs on **GitLab Runners** — GitLab-hosted (SaaS) or self-hosted, registered with a specific project/group and often selected via **tags**.
- **Rules** (`rules:`) or the older `only`/`except` control which jobs run for which branches/events, more expressive than GitHub Actions' `on:` alone.
- Deploy targets are represented as **environments**, with built-in support for **review apps** (an auto-provisioned, ephemeral environment per merge request).

## Why it exists

Teams already using GitLab for source control wanted the same "no separate service to wire up" benefit GitHub Actions later provided GitHub users — and GitLab shipped it years earlier. Its stage/job model (explicit sequential stages, parallel jobs within a stage) maps naturally onto the common build→test→deploy shape without needing an explicit `needs:` dependency graph for the common case (though `needs:` is available for more advanced fan-out). Its deeper "one application" integration (built-in registry, environments, review apps, security scanning) reduces the number of separate tools/services a team needs to wire together compared to composing many third-party integrations.

## How it works (high level)

1. A `.gitlab-ci.yml` file defines `stages:` (an ordered list) and one or more `jobs`, each assigned to a stage via `stage:`.
2. A pipeline triggers on a push, merge request event, schedule, or manual trigger, per each job's `rules:`.
3. GitLab dispatches each job to an available **Runner** matching its `tags:` (if specified).
4. Jobs in the same stage run in parallel; the pipeline only advances to the next stage once all jobs in the current stage finish (unless `needs:` is used to bypass strict stage ordering).
5. A failed job typically fails its stage and blocks later stages (configurable per job with `allow_failure`).
6. **Environments** track what's deployed where; **review apps** can spin up a temporary environment per merge request for manual QA before merge.
