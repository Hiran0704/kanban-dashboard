# Kanban Dashboard – Docker & Jenkins CI/CD on AWS

A DevOps project demonstrating how to containerize a React application, publish Docker images, and automate deployment to an AWS EC2 instance using Jenkins and a GitHub webhook.

## Project Overview

This project uses a Kanban Dashboard application built with React, TypeScript, and Vite. The application is packaged into a Docker container using a multi-stage Docker build and served through Nginx.

A Jenkins pipeline automates the process of checking out source code, building the Docker image, publishing it to Docker Hub, deploying it on an EC2 instance, and validating application health.

## Technology Stack

- **Frontend:** React, TypeScript, Vite
- **Containerization:** Docker
- **Web Server:** Nginx
- **CI/CD:** Jenkins
- **Source Control:** Git, GitHub
- **Container Registry:** Docker Hub
- **Cloud Infrastructure:** AWS EC2
- **Health Validation:** Docker HEALTHCHECK and HTTP health endpoint

## Architecture and Workflow

1. Developer pushes changes to GitHub.
2. GitHub webhook triggers Jenkins.
3. Jenkins checks out the latest source code.
4. Jenkins builds the Docker image.
5. Jenkins authenticates with Docker Hub and pushes the image.
6. Jenkins deploys the image on AWS EC2.
7. The pipeline validates the candidate and production application health.
8. If production deployment or health validation fails, the pipeline attempts to restore the previous Docker image.

## Repository

[Kanban Dashboard – GitHub](https://github.com/Hiran0704/kanban-dashboard)

## Docker Implementation

The project uses a multi-stage Dockerfile:

- The build stage uses Node.js to install dependencies and generate the production build.
- The runtime stage uses Nginx to serve the generated static files.
- The container runs as the non-root `nginx` user.
- Nginx listens on container port `8080`.
- A `/health` endpoint returns an HTTP success response.
- Docker HEALTHCHECK periodically checks the endpoint.

### Build the Docker image

```bash
docker build -t kanban-dashboard .
```

### Run the container locally

```bash
docker run -d \
  --name kanban-dashboard \
  -p 8080:8080 \
  kanban-dashboard
```

Open `http://localhost:8080` in your browser.

### Check container health

```bash
docker ps
docker inspect kanban-dashboard --format '{{.State.Health.Status}}'
```

## Docker Hub

The project image is published to:

[Docker Hub – hiran4024/kanban-dashboard](https://hub.docker.com/r/hiran4024/kanban-dashboard)

The Jenkins pipeline publishes images using Jenkins build-number tags and the `latest` tag.

Example:

```bash
docker pull hiran4024/kanban-dashboard:latest
```

## Jenkins CI/CD Pipeline

The Jenkins pipeline contains the following stages:

| Stage | Description |
|---|---|
| Checkout | Retrieves source code from GitHub |
| Build Docker Image | Builds the application image |
| Push to Docker Hub | Publishes the image to the registry |
| Deploy on EC2 | Deploys the application container |
| Health Check | Validates application availability |

Docker Hub authentication is configured through Jenkins credentials. Sensitive tokens and passwords are not stored in the repository.

## GitHub Webhook Integration

A GitHub webhook is configured to trigger the Jenkins pipeline when code is pushed to the repository.

The Jenkins job uses the **GitHub hook trigger for GITScm polling** option.

This enables automatic pipeline execution after repository updates without manually starting a Jenkins build.

## Deployment Validation and Rollback

The deployment pipeline uses a candidate deployment approach:

1. Records the image ID of the currently running production container.
2. Starts the new candidate container on temporary port `8081`.
3. Checks the candidate's `/health` endpoint.
4. Replaces the production container on port `80` only after the candidate passes its health check.
5. Validates the production application through its health endpoint.
6. If production deployment or health validation fails, the pipeline attempts to restore the previously running Docker image.
7. Removes the temporary candidate container after pipeline execution.

### Docker Resource Limits

Resource limits are applied to both candidate and production containers:

- **Memory:** 512 MiB
- **CPU:** 0.5 CPU

### Rollback Validation Status

The rollback mechanism is implemented in the Jenkinsfile.

The normal deployment completed successfully in Jenkins Build #13. The application was also externally accessible through the EC2 public IP after deployment.

An induced-failure rollback test has not been performed.

## AWS EC2 Deployment

The application is deployed on an AWS EC2 instance using Docker.

**Live Application URL:** http://15.252.185.146/

Deployment configuration:

- **EC2 host port:** `80`
- **Container port:** `8080`
- **Container name:** `kanban-dashboard`
- **Docker image:** `hiran4024/kanban-dashboard`

The application was successfully accessed externally through the EC2 public IPv4 address and returned an HTTP `200 OK` response.

**Note:** The public IP may change if the GUVI lab EC2 instance is recreated.

## Health Checks and Monitoring

The project uses Docker's built-in health-check mechanism and Nginx logs to validate the running application.

Check container status:

```bash
sudo docker ps
```

Check health status:

```bash
sudo docker inspect kanban-dashboard \
  --format '{{.State.Health.Status}}'
```

Check application response:

```bash
curl -I http://localhost/health
```

View container logs:

```bash
sudo docker logs --tail=30 kanban-dashboard
```

## Prerequisites

To work with or reproduce this project, you will need:

- Git and GitHub access
- Docker
- Node.js and npm for local application development
- An AWS EC2 environment
- Jenkins with Docker access
- A Docker Hub account
- A GitHub repository and webhook configuration

## Troubleshooting Notes

| Issue | Resolution used in this project |
|---|---|
| Jenkins could not access Docker | Added the Jenkins user to the Docker group and restarted Jenkins |
| Docker Hub authentication failed | Created a valid Docker Hub access token with repository read/write permissions |
| Jenkins could not find registry credentials | Configured the Docker Hub credential in Jenkins using the expected credential ID |
| Application health check failed | Checked container status, Nginx logs, and the `/health` endpoint |
| Jenkinsfile compilation failed | Corrected a Groovy comment that was mistakenly written as a shell comment |

## Project Evidence

Screenshots and supporting evidence are organized by project module:

- Application setup
- Docker containerization
- Docker security
- Container registry and versioning
- AWS infrastructure
- Jenkins configuration
- CI/CD pipeline
- GitHub webhook integration
- Health check and validation
- Logging and monitoring
- Docker resource limits
- Rollback-enabled deployment pipeline

## Project Scope

The project covers automated Docker image building, registry publishing, EC2 deployment, webhook-triggered CI/CD, candidate health validation, production health checks, rollback implementation, logging, and container resource limits.

The optional **Zero / Minimal Downtime Deployment** bonus module was intentionally skipped.

## Acknowledgements

The Kanban Dashboard application was forked from the repository provided for the GUVI project. It was used as the application source for this DevOps implementation.

The Docker containerization, Nginx configuration, Jenkins CI/CD pipeline, Docker Hub integration, AWS EC2 deployment, GitHub webhook automation, health checks, logging, resource limits, and rollback mechanism were implemented as part of this project.

---

**Author:** Hiran Aathtihya Selvamuthukumaran

**Project:** Deploy Your Code on a Docker Container using Jenkins on AWS
