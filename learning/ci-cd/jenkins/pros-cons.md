---
title: Jenkins — Pros & Cons
category: ci-cd
topic: jenkins
tags: [learning, ci-cd, jenkins, pros-cons]
related: ["[[jenkins]]"]
---

> [!success] Pros
> - **Full control, self-hosted** — runs entirely on your own infrastructure, with no dependency on a third-party platform's uptime or pricing changes. Example: an air-gapped/on-prem environment with no external network access can still run Jenkins fully internally.
> - **Massive plugin ecosystem** — thousands of community plugins cover nearly any tool or platform. Example: a legacy toolchain using a niche on-prem artifact repository still has a maintained Jenkins plugin, even though no hosted CI platform natively supports it.
> - **Not tied to any single source control host** — works equally with GitHub, GitLab, Bitbucket, or an internal Git server. Example: a company self-hosting GitLab Community Edition (without GitLab's paid CI runners) still gets full CI/CD via Jenkins.
> - **Highly customizable via Shared Libraries** — large orgs can standardize pipeline logic across hundreds of repos. Example: a `deployToKubernetes()` shared step encodes an org's entire deploy convention, called identically from every service's Jenkinsfile.
> - **Mature and battle-tested** — two decades of production use (as Hudson, then Jenkins) means most edge cases have known solutions and plugins.

> [!warning] Cons
> - **Operational burden** — you own patching, scaling, backups, and securing the Jenkins server itself, plus every agent. Example: a security vulnerability in Jenkins core or a widely-used plugin requires the team to patch and restart the controller, unlike a hosted platform where the vendor handles this.
> - **Groovy DSL learning curve** — Scripted pipelines require real Groovy knowledge, and even Declarative pipelines have Groovy-flavored quirks (e.g. `when` blocks, `script {}` escape hatches) that trip up newcomers. Example: a seemingly simple conditional stage requires wrapping logic in a `script {}` block because Declarative syntax alone can't express it.
> - **Dated default UI** — the classic Jenkins UI is widely seen as clunky compared to modern hosted CI dashboards, even with Blue Ocean available. Example: reading a long, unstructured console log to find which of many parallel stages failed is slower than a modern platform's collapsible per-stage view.
> - **Plugin fragility** — plugins can conflict, lag behind Jenkins core updates, or be abandoned by maintainers. Example: upgrading Jenkins core breaks an unmaintained plugin the team relies on, forcing a choice between staying on an old version or losing that integration.
> - **No native hosted runners** — unlike GitHub Actions/GitLab CI/CircleCI, there's no "just use the vendor's cloud machines" option; you must provision and maintain agents yourself (or via a cloud plugin). Example: scaling up agent capacity for a traffic spike in build volume requires actively managing infrastructure, not a config toggle.
> - **Security responsibility is entirely yours** — misconfigured credentials scoping, exposed Jenkins UIs, or unpatched plugins are common real-world attack vectors. Example: an internet-exposed Jenkins instance with default/weak admin credentials is a well-documented target for compromise.
