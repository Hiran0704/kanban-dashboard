# Kanban Dashboard – Docker & Jenkins CI/CD on AWS

A DevOps project demonstrating how to containerize a React application, publish Docker images, and automate deployment to an AWS EC2 instance using Jenkins and a GitHub webhook.

## Project Overview

This project uses a Kanban Dashboard application built with React, TypeScript, and Vite. The application is packaged into a Docker container using a multi-stage Docker build and served through Nginx.

A Jenkins pipeline automates the process of checking out the source code, building the Docker image, publishing it to Docker Hub, deploying it on an EC2 instance, and validating the application health.

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

1. Developer pushes changes to the GitHub repository.
2. GitHub webhook notifies Jenkins of the push.
3. Jenkins checks out the latest source code.
4. Jenkins builds the Docker image.
5. Jenkins authenticates with Docker Hub and pushes the image.
6. Jenkins deploys the image on the AWS EC2 instance.
7. The pipeline checks the application health.

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

The pipeline publishes images using Jenkins build-number tags and the `latest` tag.

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
| Deploy on EC2 | Runs the image on the EC2 instance |
| Health Check | Validates application availability |

Docker Hub authentication is configured through Jenkins credentials. Sensitive tokens and passwords are not stored in the repository.

## GitHub Webhook Integration

A GitHub webhook is configured to trigger the Jenkins pipeline when code is pushed to the repository.

The Jenkins job uses the **GitHub hook trigger for GITScm polling** option.

This enables automatic pipeline execution after repository updates without manually starting a Jenkins build.

## AWS EC2 Deployment

The application is deployed on an AWS EC2 instance running Docker and Jenkins.

The deployment maps:

- **EC2 host port:** `80`
- **Container port:** `8080`

The application can be accessed through the EC2 instance's public address when the instance is running and its security group permits the required traffic. The public address may change when the lab environment is recreated.

## Docker Resource Limits

The deployment applies resource limits to the application container:

```bash
--memory=512m
--cpus=0.5
```

These settings limit the container to:

- **Memory:** 512 MiB
- **CPU:** 0.5 CPU

Verify the configured limits on the EC2 instance:

```bash
sudo docker inspect kanban-dashboard \
  --format 'Memory={{.HostConfig.Memory}} bytes | NanoCPUs={{.HostConfig.NanoCpus}}'
```

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

## Project Scope

The project covers automated Docker image building, registry publishing, EC2 deployment, webhook-triggered CI/CD, health validation, logging, and container resource limits.

The optional **Zero / Minimal Downtime Deployment** bonus module was intentionally skipped.

---

**Author:** Hiran Aathtihya Selvamuthukumaran  
**Project:** Deploy Your Code on a Docker Container using Jenkins on AWS