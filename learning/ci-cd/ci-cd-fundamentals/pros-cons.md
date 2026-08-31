---
title: CI/CD Fundamentals — Pros & Cons
category: ci-cd
topic: ci-cd-fundamentals
tags: [learning, ci-cd, pros-cons]
related: ["[[ci-cd-fundamentals]]"]
---

> [!success] Pros
> - **Fast feedback on breakage** — a bug is caught in minutes, while the change is still fresh in the author's head. Example: a broken unit test fails the pipeline within 2 minutes of a push, instead of being discovered during a release two weeks later.
> - **Removes manual, error-prone release steps** — the same scripted steps run identically every time. Example: a deploy script that always runs migrations before restarting the app, instead of an engineer sometimes forgetting the migration step under pressure.
> - **Enables small, frequent releases** — smaller changes are inherently lower-risk than large batched releases. Example: shipping 10 small PRs a day instead of one giant release every two weeks that bundles unrelated changes together.
> - **Reproducibility** — pipeline-as-code means the exact same process runs in every environment. Example: the pipeline that deploys to staging is the same one (with different config) used for production, so staging genuinely predicts production behavior.
> - **Confidence to refactor** — a comprehensive automated test suite run on every change makes large refactors safer. Example: a developer renames a widely-used function across 40 files, trusting the pipeline to catch anything they missed.
> - **Visibility** — pass/fail status is visible directly on the PR/commit, not buried in someone's terminal. Example: a reviewer sees a red ❌ check and knows not to approve without asking about it.

> [!warning] Cons
> - **Upfront tooling and maintenance cost** — pipelines themselves need to be written, debugged, and kept up to date. Example: a team spends a day fixing a pipeline that broke because a base Docker image was deprecated, work that has nothing to do with the product itself.
> - **Flaky tests erode trust** — a pipeline with intermittently failing tests trains developers to ignore red pipelines or blindly re-run them. Example: a test that fails 1 in 20 runs due to a race condition gets treated as "probably fine, just retry" — masking real failures too.
> - **Slow pipelines slow everyone down** — if CI takes 45 minutes, developers batch up changes or context-switch away, defeating the "fast feedback" goal. Example: a monorepo pipeline that always rebuilds and retests everything, even for a one-line docs change.
> - **Continuous Deployment requires real test coverage** — removing the manual gate is only safe if the pipeline can actually catch problems; without strong tests it just means bugs reach production faster. Example: a team enables auto-deploy-on-merge before writing meaningful tests, and now every merge is a live production experiment.
> - **Secrets and permissions are a real attack surface** — pipelines often hold powerful credentials (cloud deploy keys, registry push access). Example: a compromised third-party GitHub Action in a workflow can exfiltrate repository secrets if permissions aren't scoped tightly.
> - **Can create a false sense of safety** — a green pipeline only proves what it actually tests; gaps in test coverage or missing production-like environments still let bugs through. Example: a pipeline with no load testing passes cleanly right before a release that falls over under real traffic.
