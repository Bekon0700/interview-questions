---
title: GitHub Actions — Pros & Cons
category: ci-cd
topic: github-actions
tags: [learning, ci-cd, github-actions, pros-cons]
related: ["[[github-actions]]"]
---

> [!success] Pros
> - **Native to GitHub** — no separate account, OAuth wiring, or webhook setup; status checks appear directly on the PR. Example: opening a PR immediately shows a check run without configuring anything beyond the workflow YAML in the repo.
> - **Huge Marketplace** — most common tasks (checkout, language setup, cloud deploy, Docker build/push) are a one-line `uses:` away. Example: deploying to AWS uses `aws-actions/configure-aws-credentials` instead of hand-rolled AWS CLI auth scripting.
> - **Free tier is generous for public repos** — unlimited minutes on public repositories, a solid free allowance for private ones. Example: an open-source project runs full CI on every PR from external contributors at no cost.
> - **Tight event integration** — triggers naturally on GitHub-native events (PR opened, issue labeled, release published), not just push/schedule. Example: a workflow auto-publishes release notes when a GitHub Release is published.
> - **Reusable workflows and composite actions** reduce duplication across many repos in an org. Example: one shared `deploy-node.yml` workflow is called by 15 different service repos instead of being copy-pasted into each.

> [!warning] Cons
> - **Vendor lock-in to GitHub** — moving to GitLab or Bitbucket means rewriting the entire pipeline in a different format. Example: a company migrating source control off GitHub has to redo years of accumulated workflow YAML from scratch.
> - **YAML debugging is painful** — syntax errors and expression context bugs (`${{ }}`) often surface only after a full run fails, since there's no great local dry-run. Example: a wrong context reference for a matrix value silently evaluates to empty string instead of erroring at edit time.
> - **Third-party action supply-chain risk** — any `uses:` action runs arbitrary code with the job's permissions/secrets. Example: a popular but compromised action in the Marketplace could exfiltrate secrets from every repo using an unpinned `@v1` tag.
> - **Hosted runner minutes cost money at scale for private repos** — heavy matrix builds or long test suites can get expensive. Example: a matrix of 12 combinations running a 20-minute suite on every push burns through a lot of monthly minutes quickly.
> - **Self-hosted runner security is your responsibility** — a compromised self-hosted runner (especially on public repos accepting PRs) can be a serious risk since PR workflows can run attacker-controlled code. Example: a public repo naively running workflows with self-hosted runners on `pull_request_target` events is a well-known attack vector if not configured carefully.
