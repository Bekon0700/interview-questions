---
title: GitLab CI — Pros & Cons
category: ci-cd
topic: gitlab-ci
tags: [learning, ci-cd, gitlab-ci, pros-cons]
related: ["[[gitlab-ci]]"]
---

> [!success] Pros
> - **Native to GitLab, with a longer track record than GitHub Actions** — no separate service to wire up, and the integration has matured since 2012. Example: a team already on GitLab gets working CI just by adding `.gitlab-ci.yml`, no marketplace of third-party actions required for the basics.
> - **"One application" DevOps integration** — built-in container registry, environments, review apps, and security scanning reduce the number of separate tools to wire together. Example: pushing an image needs no separate registry account/auth setup — `$CI_REGISTRY_IMAGE` and its credentials are already provided as CI variables.
> - **Review apps** give reviewers a live, disposable environment per merge request. Example: a designer can click through a UI change in a real deployed environment before approving, not just read a diff.
> - **Protected variables/branches** give a clean, built-in model for scoping secrets to trusted branches. Example: a production deploy token is simply unavailable to any pipeline running on an untrusted feature branch, no extra configuration needed.
> - **`needs:`-based DAG pipelines** let independent jobs skip waiting on unrelated slow jobs in the same stage. Example: a fast service's deploy doesn't sit idle waiting for an unrelated slow service's tests to finish in the same stage.
> - **Auto DevOps** gives new/simple projects a working pipeline with essentially zero configuration.

> [!warning] Cons
> - **Vendor lock-in to GitLab**, same category of risk as GitHub Actions' lock-in to GitHub. Example: migrating source hosting off GitLab means rewriting all `.gitlab-ci.yml` pipelines for whatever platform comes next.
> - **Self-managed GitLab adds real operational overhead** if not using GitLab.com's SaaS offering — you're running the whole platform, not just CI. Example: a self-hosted GitLab instance requires patching, scaling, and backing up the entire GitLab application, a much bigger surface than just a CI runner.
> - **Smaller third-party integration marketplace than GitHub Actions** — GitLab CI relies more on hand-written scripts/includes than an equivalent to the GitHub Marketplace's breadth. Example: a niche SaaS integration might have a ready-made GitHub Action but require a hand-written `curl`-based script for GitLab CI.
> - **YAML `include:`/template composition can get complex** for large orgs standardizing pipelines across many projects, similar in spirit to GitHub's reusable workflows but with its own learning curve. Example: debugging a pipeline assembled from several `include:`d template files spread across different repos can be harder to trace than a single self-contained workflow file.
> - **Runner management is still your responsibility for self-hosted runners** — same tradeoff as GitHub Actions self-hosted runners or Jenkins agents. Example: scaling runner capacity for a burst of pipeline activity requires actively managing infrastructure unless using GitLab SaaS's shared runners.
