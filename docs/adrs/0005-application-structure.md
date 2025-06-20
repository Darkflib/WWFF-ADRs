---
parent: Decisions
nav_order: 5
title: 0005 Application Structure
tags:
  - architecture
  - structure
  - organization
  - modularity
  - separation of concerns
  - blueprints
  - routers
---
# Application Structure

## Context and Problem Statement

A well-organized application structure is essential for maintainability, testability, and scalability. As Python applications grow in complexity, they can become difficult to maintain without a clear structural organization. How should we standardize our application structure to ensure code is modular, maintainable, and follows separation of concerns principles?

## Considered Options

* Monolithic single-file application
* Simple package-based structure
* Model-View-Controller (MVC) architecture
* Domain-Driven Design (DDD) architecture
* Layered architecture with separation of concerns
* Microservices architecture
* Functional vs. class-based organization

## Decision Outcome

Chosen option: **Layered architecture with separation of concerns, using blueprints for Flask or routers for FastAPI**, because:

* Provides clear boundaries between different parts of the application
* Improves maintainability by isolating changes to specific layers
* Enhances testability by allowing components to be tested in isolation
* Supports incremental development and scalability
* Aligns with the principles of both Flask and FastAPI frameworks
* Enables clear API boundaries between different modules

## Details

### General Principles

1. **Separation of Concerns**
   * Each component should have a single responsibility
   * Business logic should be separate from presentation logic
   * Data access should be separate from business logic
   * Configuration should be separate from application code

2. **Modularity**
   * Code should be organized into cohesive modules with clear boundaries
   * Modules should expose well-defined interfaces
   * Dependencies between modules should be explicit and minimized

3. **Consistency**
   * Similar patterns should be used throughout the application
   * File and directory naming conventions should be consistent
   * Import and package structure should follow a consistent pattern

### Standard Project Structure

The following structure serves as a template for our Python applications:

```
myapp/
├── .github/              # GitHub workflows and actions
│   └── workflows/        # CI/CD workflows
├── .devcontainer/       # Dev container configuration for VS Code
│   ├── devcontainer.json # Dev container configuration file
│   └── Dockerfile         # Dev container Dockerfile
├── .vscode/              # VS Code settings and configurations
│   └── settings.json      # VS Code workspace settings
├── pyproject.toml         # Project metadata and dependencies
├── README.md              # Project documentation
├── .env.example           # Example environment variables
├── .gitignore             # Git ignore file
├── Dockerfile             # Container definition
├── docker-compose.yml     # Multi-container application definition
├── .pre-commit-config.yaml # Pre-commit hooks configuration
├── src/                   # Application source code
│   └── myapp/             # Main package
│       ├── __init__.py    # Package initialization
│       ├── main.py        # Application entry point
│       ├── config.py      # Application configuration
│       ├── api/           # API definition layer
│       │   ├── __init__.py
│       │   ├── routes.py  # API route registration
│       │   └── v1/        # API version 1
│       │       ├── __init__.py
│       │       ├── endpoints/  # API endpoints by resource
│       │       │   ├── __init__.py
│       │       │   ├── users.py
│       │       │   └── items.py
│       │       └── models/  # API request/response models
│       │           ├── __init__.py
│       │           ├── users.py
│       │           └── items.py
│       ├── core/          # Core business logic
│       │   ├── __init__.py
│       │   ├── services/  # Business services
│       │   │   ├── __init__.py
│       │   │   ├── user_service.py
│       │   │   └── item_service.py
│       │   └── models/    # Domain models
│       │       ├── __init__.py
│       │       ├── user.py
│       │       └── item.py
│       ├── db/            # Database layer
│       │   ├── __init__.py
│       │   ├── session.py # Database session management
│       │   ├── base.py    # Base model
│       │   └── models/    # Database models
│       │       ├── __init__.py
│       │       ├── user.py
│       │       └── item.py
│       ├── schemas/       # Pydantic schemas for validation
│       │   ├── __init__.py
│       │   ├── user.py
│       │   └── item.py
│       └── utils/         # Utility functions
│           ├── __init__.py
│           └── helpers.py
├── migrations/            # Database migrations
│   └── ...
├── tests/                 # Test suite
│   ├── __init__.py
│   ├── conftest.py        # Test fixtures
│   ├── test_api/          # API tests
│   │   ├── __init__.py
│   │   └── test_users.py
│   ├── test_core/         # Core logic tests
│   │   ├── __init__.py
│   │   └── test_user_service.py
│   └── test_db/           # Database tests
│       ├── __init__.py
│       └── test_user_model.py
└── docs/                  # Documentation
    └── ...
```

### Application Layers

Our applications follow a layered architecture with the following components:

1. **API Layer (Presentation)**
   * Responsible for handling HTTP requests and responses
   * Implements input validation using Pydantic schemas
   * Delegates business logic to the service layer
   * Formats responses according to API standards
   * Organized as Flask blueprints or FastAPI routers

2. **Service Layer (Business Logic)**
   * Implements core business logic and workflows
   * Orchestrates operations across multiple domain entities
   * Maintains business rules and invariants
   * Independent of HTTP or database concerns

3. **Domain Model Layer**
   * Defines core business entities and their behaviors
   * Implements business rules specific to individual entities
   * Uses rich domain models with behavior when appropriate

4. **Data Access Layer**
   * Handles database interactions using SQLAlchemy
   * Implements repository pattern for data access
   * Manages database connections and transactions
   * Translates between domain models and database models

5. **Utility Layer**
   * Provides common functionality used across the application
   * Implements cross-cutting concerns like logging and error handling
   * Contains helper functions and utilities

### Framework-Specific Implementations

#### Flask Implementation

For Flask applications, we organize code using blueprints for modularity:

```python
# src/myapp/api/v1/endpoints/users.py
from flask import Blueprint, jsonify, request
from myapp.core.services.user_service import UserService
from myapp.schemas.user import UserCreate, UserResponse

users_bp = Blueprint("users", __name__, url_prefix="/users")
user_service = UserService()

@users_bp.route("/", methods=["POST"])
def create_user():
    user_data = UserCreate(**request.json)
    user = user_service.create_user(user_data)
    return jsonify(UserResponse.from_orm(user).dict()), 201

@users_bp.route("/", methods=["GET"])
def get_users():
    users = user_service.get_users()
    return jsonify([UserResponse.from_orm(user).dict() for user in users])

# src/myapp/api/routes.py
from flask import Flask
from myapp.api.v1.endpoints.users import users_bp
from myapp.api.v1.endpoints.items import items_bp

def register_routes(app: Flask):
    app.register_blueprint(users_bp, url_prefix="/api/v1")
    app.register_blueprint(items_bp, url_prefix="/api/v1")
```

#### FastAPI Implementation

For FastAPI applications, we use routers for modular API organization:

```python
# src/myapp/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List

from myapp.core.services.user_service import UserService
from myapp.schemas.user import UserCreate, UserResponse
from myapp.db.session import get_db

router = APIRouter(prefix="/users", tags=["users"])
user_service = UserService()

@router.post("/", response_model=UserResponse, status_code=201)
def create_user(
    user_data: UserCreate,
    db: Session = Depends(get_db)
):
    user = user_service.create_user(user_data, db)
    return user

@router.get("/", response_model=List[UserResponse])
def get_users(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db)
):
    users = user_service.get_users(db, skip=skip, limit=limit)
    return users

# src/myapp/api/routes.py
from fastapi import FastAPI
from myapp.api.v1.endpoints import users, items

def register_routes(app: FastAPI):
    app.include_router(users.router, prefix="/api/v1")
    app.include_router(items.router, prefix="/api/v1")
```

### Service Layer Implementation

The service layer encapsulates business logic and coordinates between different parts of the system:

```python
# src/myapp/core/services/user_service.py
from typing import List, Optional
from sqlalchemy.orm import Session

from myapp.db.models.user import User as UserModel
from myapp.core.models.user import User
from myapp.schemas.user import UserCreate

class UserService:
    def create_user(self, user_data: UserCreate, db: Session) -> User:
        # Check if user already exists
        existing_user = db.query(UserModel).filter(
            UserModel.email == user_data.email
        ).first()
        
        if existing_user:
            raise ValueError("User with this email already exists")
        
        # Create new user
        db_user = UserModel(
            email=user_data.email,
            username=user_data.username,
            full_name=user_data.full_name,
            hashed_password=self._hash_password(user_data.password)
        )
        
        db.add(db_user)
        db.commit()
        db.refresh(db_user)
        
        # Convert to domain model
        return User.from_orm(db_user)
    
    def get_users(self, db: Session, skip: int = 0, limit: int = 100) -> List[User]:
        users = db.query(UserModel).offset(skip).limit(limit).all()
        return [User.from_orm(user) for user in users]
    
    def _hash_password(self, password: str) -> str:
        # Implementation of password hashing
        return "hashed_" + password  # Simplified for example
```

### SQLAlchemy Models

Database models using SQLAlchemy:

```python
# src/myapp/db/base.py
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

# src/myapp/db/models/user.py
from sqlalchemy import Boolean, Column, Integer, String, DateTime
from sqlalchemy.sql import func
from myapp.db.base import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    username = Column(String, unique=True, index=True)
    full_name = Column(String)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
```

### Pydantic Schemas

Data validation and serialization using Pydantic:

```python
# src/myapp/schemas/user.py
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, EmailStr, Field, constr

class UserBase(BaseModel):
    email: EmailStr
    username: constr(min_length=3, max_length=50)
    full_name: str

class UserCreate(UserBase):
    password: constr(min_length=8)

class UserResponse(UserBase):
    id: int
    is_active: bool
    created_at: datetime
    
    class Config:
        orm_mode = True
```

## Consequences

### Positive

* Clear separation of concerns makes the codebase easier to maintain and understand
* Modular structure enables parallel development by different team members
* Testing is simplified as components can be tested in isolation
* New features can be added with minimal impact on existing code
* Code reuse is encouraged through proper abstraction

### Negative

* More initial complexity compared to simpler structures
* Can be perceived as over-engineered for very small applications
* Requires discipline to maintain the separation between layers
* More files and directories to navigate

## 12-Factor App Methodology

Our application structure incorporates principles from the [12-Factor App methodology](https://12factor.net/), adapted for Python applications:

1. **Codebase**: One codebase tracked in version control, many deploys
   * Each application has a single repository
   * Deployment environments (dev, staging, production) use the same codebase with different configurations

2. **Dependencies**: Explicitly declare and isolate dependencies
   * Dependencies are declared in `pyproject.toml` (poetry) or `requirements.txt`
   * Use `uv` for dependency management and virtual environments
   * Ensure reproducible builds with specific versions or hashes

3. **Config**: Store configuration in the environment
   * Use environment variables for configuration that varies between environments
   * Use `pydantic` with `python-dotenv` for environment variable management
   * Never commit sensitive configuration to version control
   * Never bake sensitive data or configuration into container images
   * Use secrets management services (e.g., GCP Secret Manager) for production secrets

4. **Backing Services**: Treat backing services as attached resources
   * Use environment variables for service connection strings
   * Design for easy swapping of backing services (e.g., development SQLite vs. production PostgreSQL)

5. **Build, Release, Run**: Strictly separate build and run stages
   * Build: Convert code to executable bundle
   * Release: Combine build with configuration
   * Run: Execute the application in the environment

6. **Processes**: Execute the app as one or more stateless processes
   * Application is stateless; all persistent data stored in backing services
   * For Docker and Kubernetes deployments, persistent data may be stored in volumes
   * Avoid local file storage for session state or user uploads

7. **Port Binding**: Export services via port binding
   * Applications are self-contained and expose their services via ports
   * Use Gunicorn (Flask) or Uvicorn (FastAPI) for production deployment
   * For container-based services, consider Unix sockets for intra-pod container communication to improve efficiency

8. **Concurrency**: Scale out via the process model
   * Use Gunicorn workers for Flask applications
   * Leverage asyncio for FastAPI applications when appropriate

9. **Disposability**: Maximize robustness with fast startup and graceful shutdown
   * Applications should start quickly and shut down gracefully
   * Handle SIGTERM signals properly

10. **Dev/Prod Parity**: Keep development, staging, and production as similar as possible
    * Use containers (Docker) to ensure environment consistency
    * Use the same backing services in development and production where feasible

11. **Logs**: Treat logs as event streams
    * Write logs to stdout/stderr
    * Use structured logging with JSON format
    * Avoid writing to log files directly from the application

12. **Admin Processes**: Run admin/management tasks as one-off processes
    * Use Flask CLI or FastAPI dependencies for administrative tasks
    * Ensure admin code uses the same environment as the application code

### Python-Specific Adaptations

While the 12-factor methodology provides excellent general guidance, we adapt some principles to better fit Python best practices:

1. **Configuration Management**: While strict 12-factor suggests all configuration in environment variables, we use a hybrid approach with:
   * Pydantic `Settings` classes for type-safe configuration
   * `.env` files for local development (never committed to version control)
   * Environment variables for production deployment

2. **Package Structure**: We follow Python's packaging best practices:
   * Use `src/` layout to avoid import issues during development
   * Provide proper `__init__.py` files for package hierarchy
   * Follow PEP 8 and PEP 420 for package organization

3. **Process Management**: For Python applications, we use:
   * Gunicorn workers for Flask applications
   * Uvicorn with Gunicorn workers for FastAPI applications
   * Proper signal handling for graceful shutdowns

4. **Dependency Isolation**: We use `uv` for dependency management and virtual environments, which provides better performance and reproducibility than traditional tools like pip or virtualenv.

## Related Decisions

* [0011 Language and Framework](./0011-language-and-framework.md)
* [0012 Database](./0012-database.md)
* [0031 Local Development Environment](./0031-local-development-environment.md)
* [0032 Web Framework Selection](./0032-web-framework-selection.md)
* [0034 Data Validation](./0034-data-validation.md)
* [0036 Build Process](./0036-build-process.md)
