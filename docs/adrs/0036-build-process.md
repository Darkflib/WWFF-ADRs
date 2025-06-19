---
parent: Decisions
nav_order: 36
title: 0036 Application Build Process
tags:
  - python
  - build
  - ci
  - container
  - pipeline
  - reproducibility
---
# Application Build Process

## Context and Problem Statement

Software deployment requires a reliable, repeatable, and efficient build process. For Python applications, this involves dependency resolution, testing, linting, packaging, and containerization. How should we standardize our build process to ensure consistency, reproducibility, and automation?

## Considered Options

* Manual builds with local developer environments
* Scripted builds using shell scripts
* CI/CD pipeline builds (GitHub Actions)
* Containerized builds with multi-stage Docker builds
* Third-party build systems and platforms

## Decision Outcome

Chosen option: **Containerized builds with multi-stage Docker builds in CI/CD pipelines**, because:

* Ensures consistent build environments regardless of where the build runs
* Creates reproducible builds by eliminating "works on my machine" issues
* Supports both local development builds and automated CI/CD builds
* Enables efficient caching and layering for faster builds
* Results in production-ready container images that can be directly deployed

## Details

### Build Process Overview

Our Python application build process consists of the following stages:

1. **Dependency Resolution**
   - Use `uv` for dependency management and resolution
   - Generate a lock file for reproducible builds
   - Validate dependencies for security vulnerabilities

2. **Code Quality Checks**
   - Run linters (flake8, black in check mode)
   - Run static type checking with mypy
   - Verify import ordering with isort

3. **Testing**
   - Run unit tests with pytest
   - Generate test coverage reports with pytest-cov
   - Run integration tests where applicable

4. **Packaging**
   - Create Python packages using pyproject.toml
   - Version artifacts using semantic versioning
   - Generate documentation

5. **Containerization**
   - Use multi-stage Docker builds
   - Create minimal production images
   - Scan images for vulnerabilities

### Implementation Details

#### Dependency Management

For Python dependency management, we use `uv` to create reproducible builds:

```bash
# In CI environment
uv pip compile requirements.in -o requirements.txt
uv pip install -r requirements.txt
```

#### Docker Build Process

We use multi-stage Docker builds to separate the build environment from the runtime environment:

```dockerfile
# Build stage
FROM python:3.11-slim as builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install uv
RUN pip install uv

# Copy and install dependencies
COPY requirements.txt .
RUN uv pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Run tests and linting
RUN uv pip install pytest flake8 mypy black isort
RUN flake8 .
RUN black --check .
RUN isort --check-only --profile black .
RUN mypy .
RUN pytest

# Runtime stage
FROM python:3.11-slim

WORKDIR /app

# Copy only what's needed from the builder stage
COPY --from=builder /app /app
COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages

# Set Python path
ENV PYTHONPATH=/app

# Application configuration
ENV PORT=8080
EXPOSE ${PORT}

# Run command depends on the framework:
# For Flask applications:
CMD ["sh", "-c", "gunicorn --bind 0.0.0.0:${PORT} app:app"]
# For FastAPI applications:
# CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT}"]
```

#### CI/CD Integration

In GitHub Actions, the build process is automated using workflows:

```yaml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Build and test
        uses: docker/build-push-action@v4
        with:
          context: .
          push: false
          load: true
          tags: app:test
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: app:test
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
```

## Consequences

### Positive

* Highly reproducible builds regardless of the environment
* Clear separation between build-time and runtime dependencies
* Improved CI/CD integration with automated testing and vulnerability scanning
* Reduced "works on my machine" issues
* Consistent production container images

### Negative

* Increased complexity compared to simple script-based builds
* Requires Docker knowledge and infrastructure
* Build times can be longer due to container build overhead

### Neutral

* Requires standardizing on containerization for all applications
* Teams need to understand and follow the multi-stage Docker build pattern

## Related Decisions

* [0019 Docker](./0019-docker.md)
* [0025 CI/CD](./0025-ci-cd.md)
* [0026 Container Registry](./0026-container-registry.md)
* [0030 Python Dependency Management](./0030-python-dependency-management.md)

## Notes

The build process should be regularly reviewed and updated as new tools and best practices emerge. While this ADR establishes a standard approach, teams should adapt the specific implementation details to fit their project requirements while maintaining the core principles of reproducibility, automation, and consistency.
