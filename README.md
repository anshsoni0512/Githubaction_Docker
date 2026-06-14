# Dockerized Flask App with CI/CD Pipeline

A project demonstrating how to containerize a Python web application and automate the build, test, and publish workflow using GitHub Actions and Docker.

---

## What This Project Covers

- Writing a `Dockerfile` to containerize a Python app
- Setting up a two-job CI/CD pipeline with GitHub Actions
- Gating Docker image publishing behind automated tests
- Pushing Docker images to DockerHub using GitHub Secrets

---

## Project Structure

```
├── app.py                        # Flask application
├── test_app.py                   # pytest test suite
├── requirements.txt              # Python dependencies
├── Dockerfile                    # Container build instructions
└── .github/
    └── workflows/
        └── cicd.yaml             # GitHub Actions pipeline
```

---

## Docker

The `Dockerfile` builds a container image using `python:3.11-slim` as the base, installs dependencies, and runs the app on port `7890`.

**Build and run locally:**
```bash
docker build -t flasktest-app .
docker run -p 7890:7890 flasktest-app
```

---

## CI/CD Pipeline

The pipeline is defined in `.github/workflows/cicd.yaml` and runs on every push or pull request to `main`.

### Pipeline Flow

```
push / pull_request to main
        │
        ▼
┌─────────────────┐
│  build-and-test │  → checkout → setup Python → install deps → pytest
└────────┬────────┘
         │ passes
         ▼
┌──────────────────────┐
│  build-and-publish   │  → checkout → setup Buildx → login → build + push to DockerHub
└──────────────────────┘
```

### Job 1 — `build-and-test`

Runs on a fresh `ubuntu-latest` runner. Installs dependencies and runs the test suite with `pytest`. If any test fails, the pipeline stops here and the Docker image is never built.

### Job 2 — `build-and-publish`

Only runs if `build-and-test` passes (`needs: build-and-test`). Uses Docker Buildx to build the image and push it to DockerHub.

---

## GitHub Secrets Required

Go to **Settings → Secrets and Variables → Actions** in your GitHub repo and add:

| Secret | Value |
|---|---|
| `DOCKER_USERNAME` | Your DockerHub username |
| `DOCKER_PASSWORD` | Your DockerHub password or access token |

---

## Pulling the Image

Once the pipeline runs successfully, the image is available on DockerHub:

```bash
docker pull <your-dockerhub-username>/flasktest-app:latest
docker run -p 7890:7890 <your-dockerhub-username>/flasktest-app:latest
```
