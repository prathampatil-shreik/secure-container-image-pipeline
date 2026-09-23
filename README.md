# Secure Container Image Pipeline

A demonstration of a secure container image build pipeline using Docker multi-stage builds, vulnerability scanning with Trivy, and GitHub Actions CI/CD.

## Project Structure

```
secure-container-image-pipeline/
├── .github/workflows/       # GitHub Actions CI/CD pipeline
├── docs/screenshots/        # Pipeline screenshots
├── src/                     # Spring Boot application source
├── Dockerfile               # Secure multi-stage build
├── Dockerfile.naive         # Naive single-stage build (for comparison)
├── .dockerignore
└── pom.xml
```

## Key Concepts

- **Multi-stage build** — `Dockerfile` separates build and runtime stages, keeping the final image minimal
- **Naive build** — `Dockerfile.naive` shows the insecure single-stage approach for comparison
- **Vulnerability scanning** — Trivy scans the image for CRITICAL/HIGH CVEs in the CI pipeline
- **GitHub Actions** — Automates build, scan, and push on every push/PR to `main`

## Running Locally

```bash
docker build -t secure-app .
docker run -p 8080:8080 secure-app
```

## Pipeline

The GitHub Actions workflow (`.github/workflows/container-pipeline.yml`) will:
1. Build the Docker image
2. Scan with Trivy — fails on CRITICAL or HIGH vulnerabilities
