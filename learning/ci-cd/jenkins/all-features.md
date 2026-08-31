---
title: Jenkins — All Features
category: ci-cd
topic: jenkins
tags: [learning, ci-cd, jenkins, features]
related: ["[[jenkins]]"]
---

Comprehensive feature list for Jenkins. Each feature includes why it was necessary and a concrete example.

## Declarative vs Scripted Pipelines

**Declarative** pipelines use a structured, opinionated `pipeline { agent {} stages {} }` syntax; **Scripted** pipelines are raw Groovy with a `node {}` block, giving full programmatic control.

> [!tip] Why this was necessary
> Early Jenkins pipelines (Scripted) required real Groovy programming knowledge for even simple cases, which was a high barrier. Declarative syntax covers the common 90% of cases with a simpler, more readable, linting-friendly structure, while Scripted remains available for genuinely complex logic.

> [!example]
> ```groovy
> // Declarative
> pipeline {
>   agent any
>   stages {
>     stage('Build') { steps { sh 'npm install' } }
>     stage('Test')  { steps { sh 'npm test' } }
>   }
> }
> ```

## Agents and labels

Build work is distributed to **agents** (worker machines), selected by matching **labels** (e.g. `docker`, `windows`, `gpu`) rather than running everything on the controller itself.

> [!tip] Why this was necessary
> Running all builds directly on the controller doesn't scale and risks destabilizing the whole Jenkins instance under load; distributing work to labeled agents lets you scale horizontally and route jobs to hardware with the right capabilities.

> [!example]
> ```groovy
> pipeline {
>   agent { label 'linux && docker' }
>   ...
> }
> ```

## Plugin ecosystem

Nearly every integration (Git, Docker, Kubernetes, Slack, AWS, SonarQube) is added via an installable plugin rather than being built into core.

> [!tip] Why this was necessary
> No single vendor could keep pace building native integrations for every tool teams use; a community plugin model let Jenkins integrate with virtually anything without bloating its core.

> [!example]
> Installing the "Docker Pipeline" plugin unlocks `docker.build(...)` and `docker.image(...).inside {}` steps directly in a Jenkinsfile.

## Shared Libraries

Reusable Groovy code (custom steps, common pipeline logic) can be published as a **Shared Library**, imported by any Jenkinsfile with `@Library`.

> [!tip] Why this was necessary
> Large organizations with many Jenkins pipelines need to avoid every team hand-rolling (and independently maintaining) the same deploy/notification logic.

> [!example]
> ```groovy
> @Library('my-shared-lib') _
> deployToKubernetes(namespace: 'staging')
> ```

## Parallel stages

Stages can run concurrently instead of sequentially using a `parallel {}` block.

> [!tip] Why this was necessary
> Running independent test suites (unit, lint, a separate integration suite) one after another wastes wall-clock time when they don't depend on each other.

> [!example]
> ```groovy
> stage('Tests') {
>   parallel {
>     stage('Unit') { steps { sh 'npm run test:unit' } }
>     stage('Lint') { steps { sh 'npm run lint' } }
>   }
> }
> ```

## Post conditions

A `post {}` block runs steps based on the pipeline's final outcome (`always`, `success`, `failure`, `unstable`), regardless of which stage ran.

> [!tip] Why this was necessary
> Cleanup (removing temp files, tearing down test containers) and notifications need to happen no matter how the pipeline ended, not duplicated inside every stage.

> [!example]
> ```groovy
> post {
>   failure { slackSend(message: "Build failed: ${env.BUILD_URL}") }
>   always  { sh 'docker compose down' }
> }
> ```

## Credentials management

Jenkins stores secrets (tokens, SSH keys, passwords) encrypted in its **Credentials** store, injected into pipeline steps by ID rather than hardcoded.

> [!tip] Why this was necessary
> Same reasoning as any CI platform: secrets committed to a Jenkinsfile in source control are permanently exposed via git history.

> [!example]
> ```groovy
> withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
>   sh 'docker login -u $USER -p $PASS'
> }
> ```

## Blue Ocean

A modern, visual pipeline UI plugin providing a cleaner, more graphical view of pipeline stages and status than Jenkins' classic UI.

> [!tip] Why this was necessary
> Jenkins' original UI (a long text console log and a simple stage-view widget) was widely criticized as dated and hard to read compared to newer hosted CI dashboards; Blue Ocean addressed that gap directly.

> [!example]
> Opening a pipeline in Blue Ocean shows each stage as a connected node in a horizontal graph, color-coded green/red/yellow, with logs expandable per stage.

## Multibranch Pipelines

Jenkins automatically discovers and creates a pipeline job per branch (and per pull request) in a repository, using each branch's own Jenkinsfile.

> [!tip] Why this was necessary
> Manually creating and maintaining a separate Jenkins job for every feature branch doesn't scale; multibranch pipelines auto-manage job creation/removal as branches come and go.

> [!example]
> Opening a PR against a repo configured as a Multibranch Pipeline automatically spins up a new pipeline job scoped to that PR's branch, and removes it once the branch is deleted.
