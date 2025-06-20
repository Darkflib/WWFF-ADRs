---
parent: Decisions
nav_order: 6
title: 0006 Application Lifecycle
tags:
  - architecture
  - lifecycle
  - development
  - deployment
  - monitoring
  - maintenance
---
# Application Lifecycle

Date: 2025-06-20

## Status

Accepted

## Context

A well-defined application lifecycle is crucial for ensuring consistent development, deployment, monitoring, and maintenance of our Python applications. This ADR defines the stages of an application's lifecycle, from initialization to retirement, and establishes standards for managing each stage.

## Decision

We will adopt a comprehensive application lifecycle model with the following stages and practices:

### 1. Initialization

New Python projects are initialized with a consistent structure:

```bash
# Create project directory and basic structure
mkdir -p myproject/src/myproject
cd myproject

# Initialize git repository
git init
git branch -m main

# Create virtual environment
uv venv

# Create basic project structure if not using a cutter template
# This can be customized based on the project type
mkdir -p src/myproject/api src/myproject/core src/myproject/db tests docs
```

#### Standard Project Templates

We maintain cutter project templates for common application types:

- Flask web application
- FastAPI REST service
- CLI application
- Background worker

Each template includes:

- Standard directory structure following [0005 Application Structure](./0005-application-structure.md)
- Base `pyproject.toml` with common dependencies
- `.pre-commit-config.yaml` for code quality checks
- CI/CD pipeline configuration
- Dockerfile and docker-compose.yml
- VS Code configuration with recommended extensions

### 2. Development

During the development phase, our workflow includes:

#### Environment Setup

- Developers use the standardized local development environment defined in [0031 Local Development Environment](./0031-local-development-environment.md)
- Development containers (devcontainers) are preferred to ensure consistency
- The `.env` file contains local configuration following [0004 Configuration](./0004-configuration.md)

#### Code Development Workflow

1. Branch from `main` following our Git branching strategy ([0024 Git](./0024-git.md))
2. Implement features or fixes with continuous testing
3. Ensure code follows our Python standards:
   - Code formatting with `black`
   - Import ordering with `isort`
   - Type checking with `mypy`
   - Linting with `ruff`
4. Run tests with `pytest` and maintain coverage
5. Create pull request for review
6. Merge after approval and CI pipeline success

#### Dependency Management

- Use `uv` for dependency management
- Pin dependencies with exact versions in `pyproject.toml`
- Regularly update dependencies with security scanning

### 3. Testing

Our testing strategy includes multiple layers:

#### Unit Testing

- All business logic has corresponding unit tests
- Tests are located in the `tests` directory mirroring the source structure
- Use `pytest` as the test framework
- Aim for >80% test coverage, measured with `pytest-cov`
- Use fixtures for setup and teardown of test environments.

```python
# Example test in tests/core/test_user_service.py
from unittest.mock import Mock
import pytest
from myproject.core.services.user_service import UserService
from myproject.schemas.user import UserCreate

def test_create_user_success():
    # Arrange
    mock_db = Mock()
    mock_db.query.return_value.filter.return_value.first.return_value = None
    user_service = UserService()
    user_data = UserCreate(email="test@example.com", username="testuser", 
                          full_name="Test User", password="securepassword")
    
    # Act
    result = user_service.create_user(user_data, mock_db)
    
    # Assert
    assert result.email == "test@example.com"
    assert result.username == "testuser"
    assert mock_db.add.called
    assert mock_db.commit.called
```

#### Integration Testing

- Test interactions between components
- Use SQLite for database integration tests
- Use local containers for external service mocking

#### End-to-End Testing

- Test complete user journeys
- Use containerized application with test databases
- Automate API testing with `pytest` and `httpx`

### 4. Building

The build process creates deployable artifacts:

- Containerized applications using Docker
- The build process is defined in [0036 Build Process](./0036-build-process.md)
- CI/CD pipelines automate building on commit/PR
- Container images are versioned with Git tags

### 5. Deployment

Applications are deployed through a controlled process:

#### Deployment Environments

1. **Development** - For feature development and initial testing
2. **Staging** - Production-like environment for final testing
3. **Production** - Live environment for end users

#### Deployment Process

1. CI/CD pipeline builds and tests the application
2. Container images are pushed to registry
3. Infrastructure as Code (Terraform) provisions or updates resources
4. Container orchestration (Kubernetes or Cloud Run) deploys the application
5. Deployment validation tests run
6. Traffic is gradually shifted to new version (blue/green deployment)

```yaml
# Example Cloud Build configuration
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/${PROJECT_ID}/${_SERVICE_NAME}:${SHORT_SHA}', '.']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/${PROJECT_ID}/${_SERVICE_NAME}:${SHORT_SHA}']
- name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
  entrypoint: gcloud
  args:
  - 'run'
  - 'deploy'
  - '${_SERVICE_NAME}'
  - '--image=gcr.io/${PROJECT_ID}/${_SERVICE_NAME}:${SHORT_SHA}'
  - '--region=${_REGION}'
  - '--platform=managed'
  - '--allow-unauthenticated'
```

### 6. Operation and Monitoring

Once deployed, applications are continuously monitored:

#### Health Checks

- All applications expose a `/health` endpoint
- Health checks verify:
  - Application responsiveness
  - Database connectivity
  - External service availability
  - Resource utilization

#### Readiness and Liveness Probes

- Readiness probes determine if the application is ready to receive traffic
- Liveness probes check if the application is alive and should be restarted
- Both types of probes are configured in the container orchestration platform (e.g., Kubernetes)

#### Monitoring and Alerting

- Applications emit structured logs to stdout/stderr
- Metrics are collected via Prometheus or Cloud Monitoring
- Distributed tracing with OpenTelemetry
- Dashboards display key metrics and SLIs/SLOs
- Alerts trigger on threshold violations
- On-call rotation handles critical issues

#### Observability Standards

- **Logging**: JSON-formatted logs with standard fields:
  - Timestamp
  - Request ID
  - Service name
  - Log level
  - Message
  - Structured data

- **Metrics**: Standard application metrics:
  - Request count/rate
  - Error rate
  - Request duration (p50, p95, p99)
  - Resource utilization
  - Business metrics specific to application domain

- **Tracing**: Trace sampling based on traffic volume

### 7. Maintenance and Updates

Regular maintenance activities include:

- Dependency updates (monthly)
- Security patch application (immediate for critical issues)
- Performance optimization based on metrics
- Scheduled non-breaking changes
- Technical debt reduction

### 8. Scaling

Applications are designed to scale based on demand:

- Horizontal scaling through containerization
- Load balancing across instances
- Database connection pooling
- Cache utilization for frequent queries
- Asynchronous processing for CPU-bound tasks

### 9. Retirement

When an application reaches end-of-life:

1. Develop migration plan for users/data
2. Communicate retirement schedule
3. Implement redirects or graceful degradation
4. Archive code and documentation
5. Remove infrastructure and DNS entries
6. Document lessons learned

## Consequences

### Positive

- Standardized lifecycle ensures consistency across projects
- Clear processes improve developer productivity
- Automated testing and deployment reduce errors
- Monitoring helps identify issues proactively
- Maintainable applications through consistent practices

### Negative

- Process overhead for very small applications
- Learning curve for developers new to the team
- Maintenance of tooling and infrastructure required

## Related Decisions

* [0004 Configuration](./0004-configuration.md)
* [0005 Application Structure](./0005-application-structure.md)
* [0007 Application Deployment](./0007-application-deployment.md)
* [0008 Application Monitoring](./0008-application-monitoring.md)
* [0024 Git](./0024-git.md)
* [0025 CI/CD](./0025-ci-cd.md)
* [0031 Local Development Environment](./0031-local-development-environment.md)
* [0036 Build Process](./0036-build-process.md)
