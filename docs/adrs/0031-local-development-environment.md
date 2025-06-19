---
parent: Decisions
nav_order: 31
title: 0031 Local Development Environment
tags:
  - python
  - development
  - environment
  - devcontainer
  - wsl
  - uv
  - dotenv
---
# Local Development Environment

## Context and Problem Statement

Consistent development environments are crucial for team productivity, code quality, and reducing "works on my machine" issues. How do we standardize local development environments across our team, particularly considering the challenges of cross-platform development with Windows, macOS, and Linux?

## Considered Options

* Individual manual setup with documentation
* Virtual machines (VMs) for development
* Containerized development environments (dev containers)
* Windows Subsystem for Linux (WSL)
* Virtual environments with different tools (venv, pipenv, poetry, uv)
* Various approaches to environment variable management

## Decision Outcome

Chosen option: **Dev containers as primary approach, with WSL for Windows users, uv for Python dependency management, and dotenv for environment variables**, because:

* Dev containers provide consistent, reproducible environments across all operating systems
* WSL offers a Linux-compatible environment for Windows users when containers aren't viable
* uv provides fast, reliable dependency management and virtual environment creation
* Dotenv files allow for easy management of environment-specific configuration
* This combination minimizes cross-platform issues while maximizing developer productivity

## Details

### Development Environment Setup

#### Primary Approach: Dev Containers

Dev containers are the preferred approach for development environments, especially for new team members and complex projects:

1. **Requirements**:
   * Docker Desktop (Windows/macOS) or Docker Engine (Linux)
   * Visual Studio Code with Remote - Containers extension

2. **Standard Configuration**:
   * All projects should include a `.devcontainer` directory with:
     * `devcontainer.json` - Container configuration
     * `Dockerfile` - Container image definition
     * Optional supporting files (e.g., `docker-compose.yml`)

3. **Example Configuration**:

```json
// devcontainer.json
{
  "name": "Python Development",
  "build": {
    "dockerfile": "Dockerfile",
    "context": ".."
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-python.black-formatter",
        "matangover.mypy",
        "njpwerner.autodocstring"
      ],
      "settings": {
        "python.formatting.provider": "black",
        "python.linting.enabled": true,
        "python.linting.mypyEnabled": true,
        "editor.formatOnSave": true
      }
    }
  },
  "forwardPorts": [5000],
  "postCreateCommand": "uv pip install -e '.[dev]'",
  "remoteUser": "vscode"
}
```

```dockerfile
# Dockerfile
FROM python:3.11-slim

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# Install uv
RUN pip install uv

# Create a non-root user
ARG USERNAME=vscode
ARG USER_UID=1001 # Default user ID for vscode is 1000, but causes conflicts on some images due to existing gid 1000, so we use 1001
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME

# Set the working directory
WORKDIR /workspace

# Switch to the non-root user
USER $USERNAME
```

#### Alternative: Windows Subsystem for Linux (WSL)

For Windows users who cannot use dev containers, WSL is the recommended alternative:

1. **Setup**:
   * Install WSL 2 with Ubuntu as the preferred Linux distribution
   * Install Python 3.11+ within WSL
   * Install uv for dependency management

2. **Best Practices**:
   * Store code within the WSL filesystem, not the Windows filesystem
   * Use VS Code's Remote - WSL extension to work with code in WSL
   * Maintain the same tooling as in dev containers

3. **Command to install uv in WSL**:

```bash
# Update and install Python
sudo apt update
sudo apt install python3-pip python3-venv -y

# Install uv
pip3 install uv
```

### Python Environment Management

We use uv for virtual environment management and dependency resolution:

1. **Virtual Environment Creation**:

```bash
# Create a virtual environment in .venv directory
uv venv
```

2. **Running Commands in the Virtual Environment**:

```bash
# Preferred method (no activation needed)
uv run python script.py

# Alternative: activate the virtual environment
. .venv/bin/activate
python script.py
```

3. **Dependency Management**:

```bash
# Install dependencies from pyproject.toml
uv pip install -e ".[dev]"
# or
uv sync --group dev

# Add a new dependency
uv pip install requests

# Generate lock file (if not using pyproject.toml)
uv pip compile requirements.in -o requirements.txt

# Install from lock file
uv pip install -r requirements.txt
```

### Environment Variable Management

We use dotenv files for environment variable management:

1. **Standard Practice**:
   * Include a `.env.example` file in the repository with placeholder values
   * Add `.env` to `.gitignore` to prevent committing actual environment values
   * Document all required environment variables in the README.md

2. **Example .env.example File**:

```
# Application configuration
APP_NAME=MyApp
DEBUG=False

# Database configuration
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp

# API configuration
API_KEY=your_api_key_here
```

3. **Loading Environment Variables**:

```python
# In Python code
from dotenv import load_dotenv

load_dotenv()  # Loads variables from .env file
```

### Project Setup Automation

To ensure consistent project setup, each project should include:

1. **A detailed README.md with**:
   * Setup instructions for both dev container and manual approaches
   * Required dependencies and tools
   * Common commands and workflows

2. **Makefile or scripts for common tasks**:

```makefile
.PHONY: setup test lint format run

setup:
	uv pip install -e ".[dev]"

test:
	uv run pytest --cov=myapp tests/

lint:
	uv run flake8 myapp tests
	uv run mypy myapp tests

format:
	uv run black myapp tests
	uv run isort myapp tests

run:
	uv run python -m myapp
```

** VS Code Tasks** can also be defined for common commands:

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Setup",
      "type": "shell",
      "command": "uv pip install -e '.[dev]'",
      "problemMatcher": []
    },
    {
      "label": "Test",
      "type": "shell",
      "command": "uv run pytest --cov=myapp tests/",
      "problemMatcher": []
    },
    {
      "label": "Lint",
      "type": "shell",
      "command": "uv run flake8 myapp tests && uv run mypy myapp tests",
      "problemMatcher": []
    },
    {
      "label": "Format",
      "type": "shell",
      "command": "uv run black myapp tests && uv run isort myapp tests",
      "problemMatcher": []
    },
    {
      "label": "Run Application",
      "type": "shell",
      "command": "uv run python -m myapp",
      "problemMatcher": []
    }
  ]
}
```

3. **VS Code workspace settings**:

```json
// .vscode/settings.json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.formatting.provider": "black",
  "python.linting.enabled": true,
  "python.linting.mypyEnabled": true,
  "editor.formatOnSave": true,
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.codeActionsOnSave": {
      "source.organizeImports": true
    }
  }
}
```

## IDE Configuration

Visual Studio Code is our recommended IDE with the following extensions:

1. **Essential Extensions**:
   * Python (ms-python.python)
   * Pylance (ms-python.vscode-pylance)
   * Black Formatter (ms-python.black-formatter)
   * Remote - Containers (when using dev containers)
   * Remote - WSL (for Windows users with WSL)

2. **Recommended Extensions**:
   * mypy (matangover.mypy)
   * isort (ms-python.isort)
   * autoDocstring (njpwerner.autodocstring)
   * GitLens (eamodio.gitlens)
   * Docker (ms-azuretools.vscode-docker)

## Consequences

### Positive

* Consistent environments across the team reduce "works on my machine" issues
* Onboarding new developers is faster and more reliable
* Dev containers provide isolation from host system dependencies
* WSL provides a good alternative for Windows users
* uv offers faster dependency resolution than pip/poetry
* Environment separation prevents accidental production settings in development

### Negative

* Initial setup has a learning curve, especially for developers new to containers or WSL
* Dev containers require more system resources than simple virtual environments
* Some Windows-specific tools may not work within WSL or containers

## Related Decisions

* [0018 Python Standards](./0018-python-standards.md)
* [0019 Docker](./0019-docker.md)
* [0030 Python Dependency Management](./0030-python-dependency-management.md)

## Notes

While dev containers are our preferred approach, we recognize that individual preferences and project requirements may vary. The key principles of environment consistency, reproducibility, and proper isolation should be maintained regardless of the specific tools used.

For Windows users specifically, WSL performance can be improved by:

1. Adding the following to `.wslconfig` in the Windows home directory:
```
[wsl2]
memory=8GB
processors=4
```

2. Storing git repositories in the WSL filesystem rather than mounting Windows filesystem
3. Using VS Code's WSL extension rather than running VS Code from Windows and accessing WSL files directly
