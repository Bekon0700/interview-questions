---
title: GitLab CI — All Features
category: ci-cd
topic: gitlab-ci
tags: [learning, ci-cd, gitlab-ci, features]
related: ["[[gitlab-ci]]"]
---

Comprehensive feature list for GitLab CI. Each feature includes why it was necessary and a concrete example.

## Stages and jobs

Jobs are grouped into ordered **stages**; jobs within one stage run in parallel, and the pipeline only proceeds to the next stage once the current one finishes.

> [!tip] Why this was necessary
> Most pipelines naturally have a sequential shape (build must finish before test, test before deploy) with parallelism only within a phase — stages model this directly without requiring an explicit dependency graph for the common case.

> [!example]
> ```yaml
> stages: [build, test, deploy]
> unit-tests:
>   stage: test
>   script: [npm run test:unit]
> lint:
>   stage: test
>   script: [npm run lint]
> # unit-tests and lint run in parallel; both must finish before "deploy" stage starts
> ```

## `rules:`

Fine-grained conditions controlling whether a job runs, based on branch, event type, variables, or file changes.

> [!tip] Why this was necessary
> Real pipelines need more nuance than "run on push" — e.g. only deploy on `main`, only run e2e tests if certain files changed, skip a job entirely on draft merge requests.

> [!example]
> ```yaml
> deploy:
>   stage: deploy
>   script: [./deploy.sh]
>   rules:
>     - if: '$CI_COMMIT_BRANCH == "main"'
> ```

## `needs:` (DAG pipelines)

Lets a job start as soon as its specific dependencies finish, instead of waiting for its entire stage to complete — turning the pipeline into a directed acyclic graph rather than strict stage-by-stage execution.

> [!tip] Why this was necessary
> Strict stage ordering can leave a fast job in a later stage waiting on a slow, unrelated job in the current stage that it doesn't actually depend on — `needs:` removes that artificial bottleneck.

> [!example]
> ```yaml
> deploy-fast-service:
>   stage: deploy
>   needs: ['build-fast-service']   # doesn't wait for build-slow-service too
> ```

## Caching and artifacts

`cache:` persists files (e.g. dependencies) between pipeline runs for speed; `artifacts:` passes specific files from one job to a later job in the same pipeline.

> [!tip] Why this was necessary
> Caching avoids redundant work (reinstalling dependencies); artifacts solve a different problem — explicitly passing a build's output (e.g. a compiled bundle) to the deploy job that needs it, since each job runs in its own isolated environment.

> [!example]
> ```yaml
> build:
>   script: [npm run build]
>   artifacts:
>     paths: [dist/]
> deploy:
>   script: [./deploy.sh dist/]   # dist/ is available here because of "artifacts" above
> ```

## Environments and review apps

`environment:` tracks what's deployed where; a merge request can auto-provision a temporary **review app** environment for manual QA, torn down automatically when the MR closes.

> [!tip] Why this was necessary
> Reviewing a change by reading a diff alone misses visual/UX issues; a live, disposable environment per merge request lets reviewers actually click through the change before approving.

> [!example]
> ```yaml
> review:
>   stage: deploy
>   script: [./deploy-review.sh]
>   environment:
>     name: review/$CI_COMMIT_REF_SLUG
>     url: https://$CI_COMMIT_REF_SLUG.review.example.com
>     on_stop: stop_review
> ```

## Auto DevOps

A set of pre-built pipeline templates GitLab can apply automatically based on detected project type, covering build, test, security scanning, and deploy with minimal configuration.

> [!tip] Why this was necessary
> Many projects share a common enough shape (build a container, run tests, deploy) that hand-writing a full `.gitlab-ci.yml` from scratch for every new project is unnecessary boilerplate.

> [!example]
> Enabling Auto DevOps on a Dockerized project automatically builds an image, runs Auto Test, and deploys to a Kubernetes cluster without the team writing any `.gitlab-ci.yml` themselves.

## Built-in Container Registry integration

Every GitLab project gets its own container registry by default, with predictable image paths and CI variables for authentication already provided.

> [!tip] Why this was necessary
> Wiring up authentication to a separate third-party registry (Docker Hub, ECR) for every project adds setup friction; a registry built into the same platform as source control and CI removes an entire integration step.

> [!example]
> ```yaml
> build:
>   script:
>     - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
>     - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
> ```

## Merge request pipelines and pipelines for merged results

Pipelines can run specifically in the context of a merge request (`rules: if: $CI_PIPELINE_SOURCE == "merge_request_event"`), including an option to test the *merged* result of source + target branch before it's actually merged.

> [!tip] Why this was necessary
> Testing only the source branch in isolation can pass CI while still breaking when merged into a target branch that has since diverged — "pipelines for merged results" catches that class of bug before merge.

> [!example]
> Enabling "Pipelines for merged results" on a project means a merge request's pipeline actually tests a temporary merge commit of source+target, not just the source branch alone.

## Protected branches and protected variables

Certain branches/tags can be marked **protected**, restricting who can push to them and which CI/CD variables (e.g. production secrets) are exposed only to jobs running on protected refs.

> [!tip] Why this was necessary
> A pipeline running on an arbitrary feature branch shouldn't have access to production deploy credentials — protected variables ensure sensitive secrets are only ever injected into jobs running on trusted, protected branches.

> [!example]
> A `PROD_DEPLOY_TOKEN` CI/CD variable marked "Protected" is only available to jobs triggered from the protected `main` branch, never from an arbitrary contributor's feature branch.
