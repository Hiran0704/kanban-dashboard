# Kanban Dashboard – Docker & Jenkins CI/CD on AWS

## 1. Project Overview

This project demonstrates the deployment of a React-based Kanban Dashboard application using Docker, Jenkins, GitHub, Docker Hub, and AWS EC2.

The objective is to implement an automated CI/CD pipeline that builds, versions, pushes, and deploys Docker images whenever changes are pushed to the GitHub repository.

The application is containerized using a multi-stage Docker build and served through Nginx.

## 2. Live Application

**Public Application URL:** http://15.252.185.146/

**GitHub Repository:** https://github.com/Hiran0704/kanban-dashboard

**Docker Hub Repository:** https://hub.docker.com/r/hiran4024/kanban-dashboard

The application is hosted on an AWS EC2 instance and served through Docker and Nginx.

## 3. Technology Stack

| Category | Technologies |
|---|---|
| Frontend | React, TypeScript, Vite |
| Containerization | Docker |
| Web Server | Nginx |
| CI/CD | Jenkins |
| Source Control | Git, GitHub |
| Container Registry | Docker Hub |
| Cloud Platform | AWS EC2 |
| Operating System | Linux |
| Security Scanning | Trivy |

## 4. System Architecture

The project follows this automated deployment workflow:

```text
Developer
    |
    v
GitHub Repository
    |
    | GitHub Webhook
    v
Jenkins Pipeline
    |
    v
Checkout Source Code
    |
    v
Build Docker Image
    |
    v
Tag Image with Build Number
    |
    v
Push Image to Docker Hub
    |
    v
AWS EC2 Instance
    |
    v
Deploy Docker Container
    |
    v
Health Check
    |
    v
Live Kanban Dashboard
```

## 5. Docker Implementation

The application is containerized using a multi-stage Dockerfile.

### Build Stage
- Uses Node.js Alpine.
- Installs project dependencies.
- Builds the React application.

### Runtime Stage
- Uses Nginx Alpine.
- Copies the production build files.
- Serves the application through Nginx.
- Runs using a non-root Nginx user.
- Includes a Docker HEALTHCHECK instruction.

### Additional Docker Files

- `Dockerfile` – Multi-stage container build.
- `nginx.conf` – Nginx server configuration.
- `.dockerignore` – Excludes unnecessary files from the Docker build context.

## 6. Docker Security

The project incorporates the following container security practices:

- Multi-stage Docker build to separate build-time and runtime dependencies.
- Lightweight Alpine-based images.
- Non-root container execution using `USER nginx`.
- `.dockerignore` to exclude unnecessary files and environment files.
- No application secrets intentionally included in the image.

### Non-root Execution Verification

The running container was inspected using:

```bash
sudo docker inspect kanban-dashboard --format '{{.Config.User}}'
```

Runtime verification:

```bash
sudo docker exec kanban-dashboard id
```

Verified output:

```text
uid=101(nginx) gid=101(nginx) groups=101(nginx)
```

### Trivy Security Scan

Trivy was used to scan the versioned Docker image:

```bash
trivy image --scanners vuln hiran4024/kanban-dashboard:13
```

Secret scanning:

```bash
trivy image --scanners secret hiran4024/kanban-dashboard:13
```

**Scan results:**

- Secret scan: 0 secrets detected.
- Vulnerability scan: One Medium-severity finding identified in the `zlib` library.
- CVE: `CVE-2026-4738`
- Installed version: `1.3.2-r1`
- Fixed version reported by Trivy: `1.4-r0`

The identified vulnerability has been documented as a known security finding. The image has not been rebuilt with the suggested package update, so this finding remains outstanding.

## 7. Docker Hub – Image Registry and Versioning

The Docker image is published to Docker Hub:

**Repository:** `hiran4024/kanban-dashboard`

Images are tagged using Jenkins build numbers to maintain version history.

Example:

```bash
docker build -t hiran4024/kanban-dashboard:13 .
docker push hiran4024/kanban-dashboard:13
```

Versioned image pull on EC2:

```bash
sudo docker pull hiran4024/kanban-dashboard:13
```

The versioned image was successfully pulled and verified on the EC2 instance.

## 8. Jenkins CI/CD Pipeline

Jenkins automates the application build and deployment process.

The pipeline includes the following stages:

1. **Checkout:** Retrieves application source code from GitHub.
2. **Build Docker Image:** Builds the Docker image.
3. **Push to Docker Hub:** Authenticates using Jenkins-managed credentials and pushes the versioned image.
4. **Deploy on EC2:** Deploys the newly built container.
5. **Health Check:** Verifies application availability after deployment.

The pipeline also applies Docker resource limits:

- Memory limit: 512 MB
- CPU limit: 0.5 CPU

Docker Hub credentials are stored securely in Jenkins Credentials rather than hardcoded in the Jenkinsfile.

## 9. GitHub Webhook Integration

GitHub webhook integration enables automatic Jenkins pipeline execution whenever changes are pushed to the repository.

Webhook configuration:

- Payload URL: `http://15.252.185.146:8080/github-webhook/`
- Content type: `application/json`
- Event: Push events

The webhook integration was verified through successful GitHub webhook delivery and automatically triggered Jenkins builds.

## 10. Deployment and Rollback Strategy

The Jenkins pipeline implements a candidate deployment and health-check-based deployment strategy.

### Deployment Process

1. Builds and publishes a new versioned Docker image.
2. Starts a candidate container on port 8081.
3. Performs an application health check on the candidate deployment.
4. Promotes the candidate after successful validation.
5. Checks the production application after deployment.

### Rollback Implementation

The Jenkinsfile includes rollback logic intended to restore the previous production container if the new deployment or its health check fails.

This helps prevent an unsuccessful deployment from leaving the application unavailable.

**Note:** The rollback logic is implemented in the pipeline, but an intentional production failure test has not been performed. The normal deployment and health-check workflow has been successfully verified.

## 11. Application Health Checks

Docker HEALTHCHECK is configured to monitor application availability.

Health verification:

```bash
sudo docker ps
```

```bash
sudo docker inspect kanban-dashboard --format '{{.State.Health.Status}}'
```

HTTP response verification:

```bash
curl -I http://localhost/
```

Expected response:

```text
HTTP/1.1 200 OK
```

The application was verified as running and healthy on the EC2 instance.

## 12. Logging and Monitoring

Application logs can be inspected using:

```bash
sudo docker logs --tail=30 kanban-dashboard
```

Container status and configuration:

```bash
sudo docker inspect kanban-dashboard
```

Jenkins provides build history and console output for monitoring successful and failed pipeline executions.

## 13. AWS Infrastructure

The application is deployed on an AWS EC2 instance.

Infrastructure components:

- AWS EC2 instance.
- Linux operating system.
- Docker Engine.
- Jenkins automation server.
- Security Group configurations.
- Public IP access for the deployed application.

**Application endpoint:** http://15.252.185.146/

## 14. Project Repository Structure

```text
kanban-dashboard/
│
├── public/
├── src/
├── Dockerfile
├── nginx.conf
├── Jenkinsfile
├── .dockerignore
├── package.json
├── package-lock.json
└── README.md
```

## 15. Project Evidence

Screenshots and implementation evidence are organised by project module.

Evidence includes:

- Application setup and build.
- Docker image creation and execution.
- Non-root container verification.
- Trivy vulnerability and secret scanning.
- Docker Hub image tags.
- Versioned image pull on EC2.
- Jenkins pipeline execution.
- GitHub webhook integration.
- Application health checks.
- Container logs and inspection.
- Docker resource limit verification.
- Public application access.

## 16. Conclusion

This project demonstrates an end-to-end Docker-based CI/CD implementation using Jenkins and AWS EC2.

It covers application containerization, image versioning, registry integration, automated deployment, health checks, container security practices, resource limits, logging, and deployment rollback logic.

The implementation provides practical experience with DevOps tools and automated application delivery workflows.

## 17. Acknowledgements

The application source was forked from the repository provided as part of the GUVI DevOps project assignment and adapted for Docker containerization and CI/CD deployment.
