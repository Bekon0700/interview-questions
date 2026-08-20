# CI/CD — Questions

Your CV mentions Docker/deployment work; expect CI/CD pipeline questions as a natural follow-up even though it's not explicitly listed.

---

## Beginner

1. What is CI/CD? What's the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?
2. Why is "pipeline as code" preferred over configuring a pipeline by hand in a UI?
3. What is a CI/CD pipeline stage/job, and what's a typical sequence of stages?
4. What is a build/CI runner (or agent)?
5. What triggers can start a pipeline (push, PR, schedule, manual)?
6. What is a build artifact?

## Intermediate

7. How do you configure a basic GitHub Actions / GitLab CI workflow to run tests on every push and pull request?
8. What is a build matrix and why would you use one?
9. How does dependency caching work in a CI pipeline, and what determines a cache hit vs miss?
10. How do you manage secrets (API keys, cloud credentials) in a pipeline without committing them to the repo?
11. What is a required status check / branch protection rule, and why does it matter?
12. How do you structure a pipeline to build a Docker image and push it to a registry (ECR, Docker Hub, GHCR)? (ties to your Docker work)
13. What's the difference between a monorepo pipeline that rebuilds everything vs one that only builds what changed?

## Advanced

14. How do you design a deployment pipeline with a manual approval gate before production?
15. Explain blue-green vs canary deployment strategies. How would you implement either in a CI/CD pipeline?
16. How do you handle rollbacks in an automated deployment pipeline?
17. How would you design CI/CD for a Node/Next.js app with staging and production environments end-to-end (build → test → containerize → deploy)? (your CV: deployment story)
18. How do you keep CI pipelines fast as a codebase and test suite grow?
19. What are the security risks of third-party actions/plugins in a pipeline, and how do you mitigate them?
20. How do you handle database migrations safely as part of an automated deployment?
