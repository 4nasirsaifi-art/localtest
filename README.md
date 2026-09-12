# Dockerized Nginx App with Automated CI/CD & Zero-Touch Deployment

A lightweight Nginx web app with a fully automated pipeline: every code push triggers a Docker image build, pushes it to DockerHub, and the running container auto-updates itself — no manual server intervention needed.

## Overview

This project demonstrates a complete CI/CD workflow using GitHub Actions for build automation and Watchtower for continuous deployment. It solves a common gap in beginner DevOps projects: most tutorials stop at "build and push an image," but never show how the running application actually picks up the new version. This pipeline closes that gap.

## Architecture / Flow

```
Developer pushes code
        │
        ▼
GitHub Actions triggered
        │
        ├── Checkout code
        ├── Login to DockerHub
        ├── Build Docker image
        └── Push image to DockerHub
        │
        ▼
Watchtower (running on host) polls DockerHub every 30s
        │
        ▼
New image detected
        │
        ├── Stops old container
        ├── Pulls new image
        └── Runs new container automatically
        │
        ▼
Updated app live — zero manual steps
```

## Tech Stack

- **Nginx (Alpine)** — lightweight web server serving static content
- **Docker** — containerization
- **GitHub Actions** — CI pipeline (build + push automation)
- **DockerHub** — container image registry
- **Watchtower** — automatic container update/deployment on new image detection

## Project Structure

```
.
├── index.html                     # Static content served by Nginx
├── Dockerfile                     # Image build instructions
└── .github/
    └── workflows/
        └── docker-build.yml       # CI pipeline definition
```

## How It Works

1. **Build stage (`Dockerfile`)**
   Uses `nginx:alpine` as a base image and copies `index.html` into Nginx's default serving directory.

2. **CI pipeline (`.github/workflows/docker-build.yml`)**
   On every push to `main`, GitHub Actions:
   - Checks out the repository
   - Authenticates with DockerHub using repository secrets
   - Builds the Docker image
   - Pushes the image to DockerHub under a fixed tag (`latest`)

3. **Continuous deployment (Watchtower)**
   Watchtower runs as a separate container on the host machine, watching the deployed container. It polls DockerHub on a fixed interval and, on detecting a new image digest, automatically stops the running container, pulls the latest image, and starts a fresh container — with no manual commands required.

## Setup & Run

**1. Clone the repo**
```bash
git clone https://github.com/4nasirsaifi-art/localtest.git
cd localtest
```

**2. Configure GitHub Secrets**
In repo Settings → Secrets and variables → Actions, add:
- `DOCKER_USERNAME`
- `DOCKER_PASSWORD` (DockerHub access token recommended)

**3. Push to trigger the pipeline**
```bash
git add .
git commit -m "update content"
git push origin main
```

**4. Run the app locally**
```bash
docker pull 4nasir/nginx-app:latest
docker run -d -p 8080:80 --name nginx-app 4nasir/nginx-app:latest
```

**5. Enable auto-deployment with Watchtower**
```bash
docker run -d \
  --name watchtower \
  -e DOCKER_API_VERSION=1.41 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --interval 30 \
  nginx-app
```

Visit `http://localhost:8080` to view the app. Any future push to `main` will automatically rebuild the image and update the running container within 30 seconds — no manual pull/run needed.

## Key Learnings

- Difference between a Docker **image** (blueprint) and a **container** (running instance)
- Why a container exits when its main process finishes (exit code 0 vs error exit codes)
- CI (build + push) and CD (deployment) are separate concerns — automating one doesn't automate the other
- Using Watchtower to bridge that gap for lightweight, self-hosted auto-deployment
- Resolving Docker API version mismatches between host and containerized tools

## Author

**Nasir Saifi**
[GitHub](https://github.com/4nasirsaifi-art) · [Portfolio](https://4nasirsaifi-art.github.io)
