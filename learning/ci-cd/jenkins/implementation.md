---
title: Jenkins — Implementation
category: ci-cd
topic: jenkins
tags: [learning, ci-cd, jenkins, implementation]
related: ["[[jenkins]]", "[[all-features]]"]
---

Hands-on work using a local Jenkins instance (via Docker) against a small Node.js repo. One fully worked example first, then a problem set — **problems 2+ are yours to build unaided**; only the expected final output is given.

## Local setup (for reference)

```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

Visit `http://localhost:8080`, unlock with the initial admin password (`docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`), install the suggested plugins, and create an admin user.

---

## Worked example — a Declarative pipeline job

**Goal:** create a Jenkins pipeline job pointed at a Jenkinsfile in a Node.js repo, running install + test on demand.

```groovy
// Jenkinsfile (in the repo root)
pipeline {
  agent any

  stages {
    stage('Install') {
      steps {
        sh 'npm install'
      }
    }
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished.'
    }
  }
}
```

In the Jenkins UI: **New Item → Pipeline**, name it `node-ci`, under **Pipeline** select "Pipeline script from SCM," point it at your repo's URL and branch, with script path `Jenkinsfile`. Click **Build Now**.

**Why this works:** `agent any` tells Jenkins to run this pipeline on any available agent (here, the controller itself acting as an agent in a simple local setup). Each `stage` groups related steps and shows as its own segment in the build's visual pipeline view. `sh` runs a shell command on the agent's workspace, which Jenkins has already checked the repo out into (because the job was configured as "Pipeline script from SCM"). The `post { always {} }` block runs regardless of whether Install/Test passed or failed.

Clicking **Build Now** and then the build number shows a **Console Output** ending with:

```
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] sh
+ npm test
...
[Pipeline] echo
Pipeline finished.
[Pipeline] End of Pipeline
Finished: SUCCESS
```

---

## Problem set

Build each of these yourself against your local Jenkins instance. Only the expected final output is given — no solution Jenkinsfile.

### Beginner

**1. Add a lint stage**
Add a `Lint` stage between Install and Test running `npm run lint` (add a trivial lint script if needed). Introduce a deliberate lint error and run the build.

Expected output:
```
The build's console output and stage view show the Lint stage failing (red), and Test never runs — the pipeline stops at the first failed stage.
```

**2. Webhook trigger from GitHub**
Configure the job to trigger automatically on a push to the repo (via a GitHub webhook pointed at your Jenkins instance, or GitHub's "Poll SCM" as a fallback if your Jenkins isn't publicly reachable). Push a commit.

Expected output:
```
A new build starts automatically in the Jenkins UI within moments of the push, without clicking "Build Now" manually.
```

### Intermediate

**3. Parallel stages**
Split the Test stage into two parallel sub-stages — `Unit Tests` and `Lint` — running concurrently instead of sequentially.

Expected output:
```
The Blue Ocean (or classic stage) view shows "Unit Tests" and "Lint" as two branches running side by side under one parent stage, both completing before the pipeline moves on.
```

**4. Credentials for a private registry**
Add a Jenkins credential (Manage Jenkins → Credentials) for a Docker registry username/password, and use `withCredentials` in a new stage to log in (`docker login`) without hardcoding the password in the Jenkinsfile.

Expected output:
```
The console output shows a successful "Login Succeeded" message from docker login, while the actual password value never appears in plaintext anywhere in the console log (Jenkins masks it as ****).
```

### Advanced

**5. Multibranch Pipeline**
Convert the job to a **Multibranch Pipeline** pointed at your repo. Create two branches, each with its own (slightly different) Jenkinsfile, and open a pull request from one of them.

Expected output:
```
The Jenkins UI shows one pipeline job auto-created per branch (and one for the PR), each running its own branch's Jenkinsfile independently, without you manually creating any of them.
```

**6. Shared Library step**
Create a minimal Shared Library repo with one custom step (e.g. `notifyBuildStatus()` that just echoes a message), configure it in Manage Jenkins → System, and call it from your Jenkinsfile's `post` block via `@Library`.

Expected output:
```
The console output includes the message printed by your shared library's custom step, and modifying the shared library repo alone (without touching this project's Jenkinsfile) changes what the next build prints.
```
