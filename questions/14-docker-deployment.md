# Docker & Deployment — Questions

You have the 2.1GB → 170MB story, so expect follow-ups on multi-stage builds and containerization.

---

## Beginner

1. What is Docker and what problem does it solve? What is the difference between an image and a container?
2. What is a Dockerfile? Explain common instructions (FROM, WORKDIR, COPY, RUN, CMD, EXPOSE, ENV).
3. What is the difference between `CMD` and `ENTRYPOINT`?
4. What is a container registry (Docker Hub, ECR)?
5. What is `.dockerignore` and why is it important?
6. What is the difference between a container and a virtual machine?

## Intermediate

7. What are Docker image layers and how does layer caching work? How do you order instructions to maximize caching? (your CV)
8. What is a multi-stage build and why is it useful? (your CV: image size reduction)
9. How do you reduce Docker image size? List techniques. (your CV: 2.1GB → 170MB)
10. What is Docker Compose and when do you use it?
11. How do you pass configuration/secrets into a container?
12. What are volumes and bind mounts? When use each?
13. How do you handle networking between containers?
14. What is a health check and why does it matter?

## Advanced

15. Walk through your Dockerfile for the Next.js image size optimization in detail. (your CV)
16. What is a reverse proxy (Nginx) and why put it in front of a Node app? (your CV: Nginx)
17. How do you achieve zero-downtime deployments (rolling, blue-green, canary)?
18. How would you deploy a Node/Next.js app to production (CI/CD pipeline, containers, orchestration)?
19. What is orchestration (Kubernetes) at a high level? What problems does it solve?
20. How do you handle graceful shutdown of a containerized Node service? (also in Node file)
21. How do you monitor and log a production containerized application?
22. How do you scale a containerized app horizontally and manage load?
