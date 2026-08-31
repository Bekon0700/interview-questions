---
title: Jenkins
category: ci-cd
topic: jenkins
tags: [learning, ci-cd, jenkins]
related: ["[[../ci-cd]]", "[[../ci-cd-fundamentals/ci-cd-fundamentals]]"]
---

> [!abstract] Overview
> **Jenkins** is a self-hosted, open-source automation server — the long-standing default for CI/CD before hosted platforms existed, and still widely used where teams need full control over their pipeline infrastructure. Unlike GitHub Actions or GitLab CI (built into a hosting platform), Jenkins is a standalone application you install and operate yourself, extended almost entirely through a huge **plugin ecosystem**.

## Backstory

Jenkins began as **Hudson**, created by Kohsuke Kawaguchi at Sun Microsystems in 2004. After Oracle acquired Sun in 2010 and a dispute arose over the project's governance and trademark, most of the community forked it into **Jenkins** in 2011. It predates GitHub Actions (2018), GitLab CI (2012), and most hosted CI platforms, and its plugin-based architecture (thousands of community plugins for source control, build tools, cloud providers, notifications) let it stay relevant by integrating with nearly anything, even as newer platforms shipped a more opinionated, hosted, batteries-included experience.

## What it is

- A **Jenkins controller** (the server) schedules and coordinates builds; actual build work runs on **agents** (formerly called "slaves") — machines registered with the controller, often labeled to route specific jobs to specific hardware.
- A **pipeline** is defined in a **Jenkinsfile**, written in Groovy-based DSL, either **Declarative** (structured `pipeline { stages { ... } }` syntax, easier to read) or **Scripted** (full Groovy, more flexible, more complex).
- **Plugins** extend nearly every part of Jenkins: source control integrations, build tools, cloud/container deploy targets, notification channels, UI dashboards (e.g. Blue Ocean).
- Jenkins can be triggered by **webhooks** (from GitHub/GitLab/Bitbucket), **polling** the SCM on a schedule, or **manually**.

## Why it exists

Before Jenkins/Hudson, teams either had no automated builds or relied on custom scripts triggered by cron with no shared UI, history, or plugin ecosystem. Jenkins provided a standard, extensible server for "build and test automatically," with a web UI to view build history/logs and a plugin architecture flexible enough to bolt onto virtually any existing toolchain, source control system, or deployment target — which is exactly why it became the default choice for over a decade before hosted CI platforms existed.

## How it works (high level)

1. A **Jenkinsfile** (declarative or scripted pipeline) is added to a project's repo, defining stages (e.g. Build, Test, Deploy).
2. A trigger fires — most commonly a webhook from the SCM on push/PR, or an SCM polling schedule.
3. The Jenkins **controller** schedules the pipeline run onto an available **agent** matching any required labels (e.g. `docker`, `linux`, `gpu`).
4. Each **stage** runs its steps on that agent; a failed stage typically halts the pipeline (configurable) and marks the build red.
5. **Post** conditions (`always`, `success`, `failure`) run cleanup or notification steps regardless of/depending on the outcome.
6. Build results, logs, and artifacts are stored on the controller and viewable via the Jenkins web UI (or the Blue Ocean plugin's more modern pipeline visualization).
