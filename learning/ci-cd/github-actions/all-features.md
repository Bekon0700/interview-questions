---
title: GitHub Actions — All Features
category: ci-cd
topic: github-actions
tags: [learning, ci-cd, github-actions, features]
related: ["[[github-actions]]"]
---

Comprehensive feature list for GitHub Actions. Each feature includes why it was necessary and a concrete example.

## Reusable Actions / Marketplace

Versioned, shareable units of workflow logic (`uses: owner/repo@version`), published publicly or privately in the Marketplace.

> [!tip] Why this was necessary
> Most workflows need the same handful of things — checkout code, install a language runtime, log into a cloud provider. Without a shared-action model, every team hand-writes and re-maintains the same shell scripts.

> [!example]
> ```yaml
> - uses: actions/checkout@v4
> - uses: actions/setup-node@v4
>   with: { node-version: '20' }
> ```

## Reusable workflows

An entire workflow file can be called from another workflow via `workflow_call`, passing inputs/secrets, instead of only sharing individual steps.

> [!tip] Why this was necessary
> Organizations with many repos often want one canonical "build and deploy a Node service" pipeline instead of copy-pasting the same YAML into every repo, which drifts out of sync over time.

> [!example]
> ```yaml
> # .github/workflows/deploy.yml (caller)
> jobs:
>   deploy:
>     uses: my-org/shared-workflows/.github/workflows/deploy-node.yml@main
>     with: { environment: production }
>     secrets: inherit
> ```

## Composite actions

A custom action defined as a sequence of steps in a single repo (`action.yml` with `runs.using: composite`), bundling several steps into one reusable `uses:` call.

> [!tip] Why this was necessary
> Copy-pasting the same 4-5 setup steps across many workflow files in one repo is repetitive and easy to let drift; a composite action packages them once.

> [!example]
> ```yaml
> # .github/actions/setup-project/action.yml
> runs:
>   using: composite
>   steps:
>     - uses: actions/setup-node@v4
>       with: { node-version: '20' }
>     - run: npm ci
>       shell: bash
> ```

## Matrix builds

`strategy.matrix` runs a job across every combination of listed variables in parallel.

> [!tip] Why this was necessary
> Verifying compatibility across multiple language versions or OSes one at a time wastes wall-clock time proportional to how many combinations exist.

> [!example]
> ```yaml
> strategy:
>   matrix:
>     node: [18, 20, 22]
> ```

## Caching (`actions/cache`)

Persists a directory between runs, keyed by a hash of a lockfile or similar, restored automatically on a matching key.

> [!tip] Why this was necessary
> Reinstalling all dependencies from scratch on every single run wastes minutes multiplied by hundreds of daily runs.

> [!example]
> ```yaml
> - uses: actions/cache@v4
>   with:
>     path: ~/.npm
>     key: npm-${{ hashFiles('package-lock.json') }}
> ```

## Environments and required reviewers

Named deploy targets (`environment:` on a job) that can require manual approval, restrict which branches can deploy, and hold environment-scoped secrets.

> [!tip] Why this was necessary
> Not every deploy target carries the same risk; production deploys often need a human gate that staging doesn't.

> [!example]
> ```yaml
> deploy:
>   environment: production   # requires approval configured in repo Settings → Environments
> ```

## `GITHUB_TOKEN` and permissions scoping

Every workflow run gets an auto-generated, short-lived `GITHUB_TOKEN` scoped to that repo; its permissions can (and should) be restricted per workflow/job.

> [!tip] Why this was necessary
> A default, overly broad token available to every step is a supply-chain risk — a compromised third-party action in the workflow could otherwise use it to push code, modify releases, or read other repo data it doesn't need.

> [!example]
> ```yaml
> permissions:
>   contents: read   # explicitly deny write access this workflow doesn't need
> ```

## Self-hosted runners

Instead of GitHub-hosted VMs, jobs can run on infrastructure you control, selected via labels.

> [!tip] Why this was necessary
> Some workloads need custom hardware (GPUs), access to a private network/VPC GitHub's cloud runners can't reach, or predictable cost at very high volume.

> [!example]
> ```yaml
> runs-on: [self-hosted, linux, gpu]
> ```

## Concurrency control

`concurrency:` groups cancel or queue overlapping runs of the same workflow/group instead of letting them race.

> [!tip] Why this was necessary
> Two pushes to the same PR in quick succession triggering two full pipeline runs wastes runner minutes on a run whose result is about to be superseded anyway.

> [!example]
> ```yaml
> concurrency:
>   group: ci-${{ github.ref }}
>   cancel-in-progress: true
> ```

## `workflow_dispatch` (manual trigger with inputs)

Lets a human trigger a workflow on demand from the GitHub UI/API, optionally supplying typed input parameters.

> [!tip] Why this was necessary
> Some actions (a one-off data migration, a hotfix deploy of a specific tag) shouldn't be automatic on every push — they need a deliberate, parameterized trigger.

> [!example]
> ```yaml
> on:
>   workflow_dispatch:
>     inputs:
>       version:
>         description: 'Tag to deploy'
>         required: true
> ```
