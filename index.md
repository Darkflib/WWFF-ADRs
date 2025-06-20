# ADRs

- [x] [0000](./docs/adrs/0000-template.md) - 2025-06-19 - Initial ADR template (based on the one from the [ADR GitHub repository](https://github.com/adr/madr))
- [ ] [0001](./docs/adrs/0001-IAM.md) - 2025-06-19 - IAM
- [ ] [0002](./docs/adrs/0002-logging-and-error-handling.md) - 2025-06-19 - Logging and Error Handling
- [ ] [0003](./docs/adrs/0003-telemetry.md) - 2025-06-19 - Telemetry
- [ ] [0004](./docs/adrs/0004-configuration.md) - 2025-06-19 - Configuration
- [x] [0005](./docs/adrs/0005-application-structure.md) - 2025-06-19 - Application structure
- [x] [0006](./docs/adrs/0006-application-lifecycle.md) - 2025-06-20 - Application lifecycle
- [ ] [0007](./docs/adrs/0007-application-deployment.md) - 2025-06-19 - Application deployment
- [ ] [0008](./docs/adrs/0008-application-monitoring.md) - 2025-06-19 - Application monitoring
- [ ] [0009](./docs/adrs/0009-application-security.md) - 2025-06-19 - Application security
- [ ] [0010](./docs/adrs/0010-application-testing.md) - 2025-06-19 - Application testing
- [ ] [0011](./docs/adrs/0011-language-and-framework.md) - 2025-06-19 - Language and framework
- [ ] [0012](./docs/adrs/0012-database.md) - 2025-06-19 - Database
- [ ] [0013](./docs/adrs/0013-caching.md) - 2025-06-19 - Caching
- [x] [0014](./docs/adrs/0014-messaging.md) - 2025-06-20 - Messaging
- [ ] [0015](./docs/adrs/0015-async-processing.md) - 2025-06-19 - Async processing
- [ ] [0016](./docs/adrs/0016-frontend.md) - 2025-06-19 - Frontend
- [ ] [0017](./docs/adrs/0017-api-design.md) - 2025-06-19 - API design
- [ ] [0018](./docs/adrs/0018-python-standards.md) - 2025-06-19 - Python standards
- [ ] [0019](./docs/adrs/0019-docker.md) - 2025-06-19 - Docker
- [ ] [0020](./docs/adrs/0020-kubernetes.md) - 2025-06-19 - Kubernetes
- [ ] [0021](./docs/adrs/0021-terraform.md) - 2025-06-19 - Terraform
- [ ] [0022](./docs/adrs/0022-aws.md) - 2025-06-19 - AWS
- [ ] [0023](./docs/adrs/0023-gcp.md) - 2025-06-19 - GCP
- [x] [0024](./docs/adrs/0024-git.md) - 2025-06-19 - Git
- [ ] [0025](./docs/adrs/0025-ci-cd.md) - 2025-06-19 - CI/CD
- [ ] [0026](./docs/adrs/0026-container-registry.md) - 2025-06-19 - Container registry
- [ ] [0027](./docs/adrs/0027-python-testing.md) - 2025-06-19 - Python testing
- [ ] [0028](./docs/adrs/0028-python-linting.md) - 2025-06-19 - Python linting
- [ ] [0029](./docs/adrs/0029-python-packaging.md) - 2025-06-19 - Python packaging
- [ ] [0030](./docs/adrs/0030-python-dependency-management.md) - 2025-06-19 - Python dependency management
- [x] [0031](./docs/adrs/0031-local-development-environment.md) - 2025-06-19 - Local development environment
- [ ] [0032](./docs/adrs/0032-web-framework-selection.md) - 2025-06-19 - Web framework selection criteria
- [ ] [0033](./docs/adrs/0033-http-client-standards.md) - 2025-06-19 - HTTP client standards
- [x] [0034](./docs/adrs/0034-data-validation.md) - 2025-06-19 - Data validation with Pydantic
- [ ] [0035](./docs/adrs/0035-cloud-storage.md) - 2025-06-19 - Cloud storage for non-hot data
- [x] [0036](./docs/adrs/0036-build-process.md) - 2025-06-19 - Application build process
- [x] [0037](./docs/adrs/0037-extranet.md) - 2025-06-20 - Extranet
- [x] [0038](./docs/adrs/0038-extranet-authentication.md) - 2025-06-20 - Extranet authentication
- [ ] [0039](./docs/adrs/0039-helm.md) - 2025-06-20 - Helm
- [ ] [0040](./docs/adrs/0040-fluxcd.md) - 2025-06-20 - FluxCD

## Summary

This repository contains a collection of Architectural Decision Records (ADRs) that document the architectural decisions made during the development of a software project. Each ADR is a standalone document that describes a specific decision, its context, and its consequences. Some ADRs may also include links to related decisions or external resources for further reading. Many are opinionated, but they are all based on our experience and best practices. Some ADRs may be updated or superseded as the project evolves, and new ADRs may be added to address new decisions or changes in the architecture.

### 0000 - Initial ADR Template
This ADR provides a template for documenting architectural decisions. It is based on the template from the [ADR GitHub repository](https://github.com/adr/madr).

Use this template to create new ADRs by copying it and filling in the relevant information for each decision.

### 0001 - IAM
This ADR documents the decisions made regarding Identity and Access Management (IAM) in the project. It covers aspects such as user authentication, authorization, and access control mechanisms.

It suggests using AWS IAM and GCP IAM for managing user identities and permissions, ensuring secure access to resources for internal users. For external users, it recommends using OAuth 2.0 and OpenID Connect for secure authentication and authorization using our own identity provider with support for federation with external identity providers.

### 0002 - Logging and Error Handling
This ADR outlines the logging strategy for the project. It emphasizes the importance of structured logging, log levels, and log aggregation.
It recommends using a centralized logging solution like AWS CloudWatch or GCP Stackdriver for log aggregation and analysis. The ADR also suggests using a consistent logging format (e.g., JSON) to facilitate parsing and querying logs.

Error handling is also discussed, recommending the use of structured error responses and proper error codes to ensure that errors are handled gracefully and provide meaningful feedback to users and developers.

### 0003 - Telemetry
This ADR discusses the telemetry strategy for the project, focusing on collecting and analyzing metrics to monitor application performance and health.
It recommends using tools like Prometheus and Grafana for metrics collection and visualization.
The ADR also suggests instrumenting the application code to expose relevant metrics and using distributed tracing tools like OpenTelemetry to trace requests across services.

### 0004 - Configuration
This ADR addresses the configuration management strategy for the project. It emphasizes the importance of separating configuration from code and using environment variables for sensitive information.
We have our own configuration management and flagging system, which allows us to manage application settings and feature flags dynamically. 

### 0005 - Application Structure
This ADR defines the standard structure for our Python applications, emphasizing a layered architecture with clear separation of concerns. It outlines the organization of code into distinct layers (API, Service, Domain Model, Data Access, and Utility) and provides specific implementations for both Flask and FastAPI frameworks.

The ADR incorporates principles from the 12-Factor App methodology, adapted specifically for Python applications. It addresses directory structure, package organization, and framework-specific patterns like Flask blueprints and FastAPI routers. By following this structure, we ensure maintainability, testability, and scalability of our applications.

### 0006 - Application Lifecycle
This ADR defines the complete lifecycle of our Python applications from initialization to retirement. It establishes standards and processes for each stage of an application's life:

1. **Initialization**: Standard project templates and setup procedures
2. **Development**: Environment setup, workflow, and coding standards
3. **Testing**: Multi-layered testing strategy with unit, integration, and E2E tests
4. **Building**: Containerized build process with CI/CD integration
5. **Deployment**: Environment definitions and controlled deployment processes
6. **Operation**: Health checks, monitoring, and observability standards
7. **Maintenance**: Regular updates, security patches, and technical debt reduction
8. **Scaling**: Strategies for handling increased load
9. **Retirement**: Orderly decommissioning process

The ADR ensures consistency across projects, improves developer productivity, and maintains application quality throughout its lifetime.

### 0007 - Application Deployment
This ADR discusses the deployment strategy for the application, including the use of containerization and orchestration.
It recommends using Docker for containerization and Kubernetes for orchestration to ensure scalability and reliability.
The ADR also suggests using infrastructure as code (IaC) tools like Terraform to manage the deployment infrastructure.

### 0008 - Application Monitoring
This ADR focuses on the monitoring strategy for the application, including health checks, performance monitoring, and alerting.
It recommends using tools like Prometheus and Grafana for monitoring application metrics and setting up alerts based on predefined thresholds.
The ADR also suggests implementing health checks for services to ensure they are running correctly and to facilitate automated recovery in case of failures.

### 0009 - Application Security
This ADR addresses the security measures for the application, including secure coding practices, vulnerability management, and incident response.
It recommends following security best practices such as input validation, output encoding, and secure authentication mechanisms.
The ADR also suggests using tools like OWASP ZAP for vulnerability scanning and implementing a security incident response plan to handle security breaches effectively.

### 0010 - Application Testing
This ADR outlines the testing strategy for the application, including unit testing, integration testing, and end-to-end testing.
It recommends using testing frameworks like pytest for unit testing and integration testing, and tools like Selenium for end-to-end testing.
The ADR emphasizes the importance of automated testing to ensure code quality and reliability throughout the development lifecycle.

### 0011 - Language and Framework
This ADR discusses the choice of programming language and framework for the application.
It recommends using Python as the primary language due to its simplicity, readability, and extensive ecosystem.
The ADR also suggests using frameworks like Flask or FastAPI for building web applications, as they provide a good balance between performance and ease of use.
It emphasizes the importance of using a modern web framework that supports asynchronous programming and has good community support for libraries and extensions.
Swagger or OpenAPI is recommended for API documentation, as it allows for easy generation of API documentation and client libraries.

For the frontend, it recommends using React or Vue.js for building interactive user interfaces, as they are widely adopted and have a strong community support.

### 0012 - Database
This ADR addresses the choice of database for the application, recommending PostgreSQL as the primary relational database
due to its robustness, scalability, and support for advanced features like JSONB and Vector data types.
It also suggests using Redis for caching and hot data storage to improve application performance.

### 0013 - Caching
This ADR discusses the caching strategy for the application, recommending the use of Redis as an in-memory data store for caching frequently accessed data.
It emphasizes the importance of caching to reduce database load and improve application performance.
The ADR also suggests implementing cache invalidation strategies to ensure data consistency between the cache and the database.

### 0014 - Messaging
This ADR establishes CloudEvents as our standard message format specification for all service-to-service communication. CloudEvents provides a consistent, transport-independent way to describe event data, enabling flexible and interoperable messaging across our applications.

The ADR covers multiple transport protocols:
- HTTP for synchronous communication and webhooks
- AMQP (via RabbitMQ) for reliable asynchronous messaging
- MQTT for lightweight IoT scenarios

It defines implementation patterns for Python applications, including code examples for creating and consuming CloudEvents messages across different transport mechanisms. The ADR also addresses message reliability, security, and deployment considerations, ensuring that our distributed systems communicate effectively while remaining loosely coupled.

### 0015 - Async Processing
This ADR discusses the asynchronous processing strategy for the application, recommending the use of RabbitMQ or Google Cloud Pub/Sub for handling background tasks and long-running processes.
It emphasizes the importance of decoupling synchronous and asynchronous operations to improve application responsiveness.

### 0016 - Frontend
This ADR addresses the frontend architecture, recommending the use of React or Vue.js for building interactive user interfaces.
It emphasizes the importance of responsive design and accessibility in frontend development.

### 0017 - API Design
This ADR outlines the API design principles for the application, recommending the use of RESTful APIs with JSON as the data format.
It emphasizes the importance of clear and consistent API endpoints, versioning, and documentation to ensure ease of use for API consumers.
The ADR also suggests using OpenAPI specifications for documenting APIs, which can be automatically generated and used for client code generation.

### 0018 - Python Standards
This ADR discusses the coding standards and best practices for Python development.
It recommends following PEP 8 for code style and formatting, using type hints for better code readability, and adhering to the Zen of Python principles.
The ADR also suggests using linters like flake8 and black for code formatting and style enforcement, as well as mypy for type checking.

There is a template repo for Python projects that follows these standards, which can be used as a starting point for new projects using cutter.

In addition, it is recommended to target 1-2 major Python versions for compatibility, with the current recommendation being Python 3.11 or later as of 2025-06-19. Ideally not the very latest version, to ensure stability. This ensures that the project benefits from the latest language features and performance improvements while maintaining compatibility with a wide range of environments. This should be updated as new Python versions are released. Python 3.14 is expected to bring further improvements and features in October 2025, and it is advisable to consider upgrading to it once it becomes stable.

Note: Python 3.14b3 is in beta as of 2025-06-19, and it is recommended to wait for its stable release before adopting it in production projects.

### 0019 - Docker
This ADR addresses the use of Docker for containerization, recommending the creation of Docker images for the application and its dependencies.
It emphasizes the importance of using multi-stage builds to optimize image size and improve build performance.
The ADR also suggests using Docker Compose for local development and testing, allowing developers to easily manage multi-container applications.

It also recommends using Github Registry or Google Container Registry for storing and distributing Docker images, ensuring that images are versioned and tagged appropriately with semantic versioning.

### 0020 - Kubernetes
This ADR discusses the use of Kubernetes for container orchestration, recommending the deployment of the application on a Kubernetes cluster for scalability and reliability.
It emphasizes the importance of using Kubernetes manifests for defining application resources and configurations, allowing for easy deployment and management of the application.
The ADR also suggests using Helm for managing Kubernetes applications, providing a way to package, configure, and deploy applications on Kubernetes.

### 0021 - Terraform
This ADR outlines the use of Terraform for infrastructure as code (IaC), recommending the use of Terraform to manage the deployment infrastructure.
It emphasizes the importance of using IaC to ensure consistency and reproducibility in infrastructure provisioning.
The ADR suggests using Terraform modules to encapsulate reusable infrastructure components and promote best practices in infrastructure management.

### 0022 - AWS
This ADR discusses the use of AWS as a cloud provider for the application, recommending the use of AWS services for hosting, storage, and networking.
It emphasizes the importance of using AWS IAM for managing user identities and permissions, ensuring secure access to resources.

### 0023 - GCP
This ADR addresses the use of Google Cloud Platform (GCP) as a cloud provider for the application, recommending the use of GCP services for hosting, storage, and networking.
It emphasizes the importance of using GCP IAM for managing user identities and permissions, ensuring secure access to resources.
The ADR also suggests using GCP Cloud Storage for object storage and GCP Cloud SQL for managed database services.

### 0024 - Git
This ADR discusses the use of Git for version control, recommending the use of Git branches for managing development workflows.
It emphasizes the importance of using Git tags for versioning releases and maintaining a clean commit history.
The ADR also suggests using Git hooks for enforcing coding standards and automating tasks like running tests and linting before commits using pre-commit hooks.

We use GitHub as our primary Git hosting platform, which provides features like pull requests, code reviews, and issue tracking to facilitate collaboration and code quality.

We use main and develop branches for managing our development workflow, with feature branches created for new features and bug fixes with the naming convention `feature/<feature-name>` or `bugfix/<bugfix-name>`.

### 0025 - CI/CD
This ADR outlines the use of continuous integration and continuous deployment (CI/CD) practices for automating the application lifecycle.
It recommends using GitHub Actions for automating build, test, and deployment processes, allowing for quick feedback and faster development cycles.
The ADR emphasizes the importance of running tests and linting checks as part of the CI/CD pipeline to ensure code quality and reliability.
It also suggests using deployment strategies like blue-green deployments or canary releases to minimize downtime and reduce risk during deployments.
However we aim to split the deploymeny and release processes, so that we can deploy the application without releasing it to users, allowing for testing and validation before the actual release.

### 0026 - Container Registry
This ADR discusses the use of container registries for storing and distributing Docker images.
It recommends using GitHub Container Registry or Google Container Registry for storing Docker images, ensuring that images are versioned and tagged appropriately with semantic versioning.
The ADR emphasizes the importance of using private container registries for security and access control, allowing only authorized users to pull and push images.
It also suggests using automated image scanning tools to detect vulnerabilities in Docker images before deployment, ensuring that only secure images are used in production environments.

### 0027 - Python Testing
This ADR outlines the testing strategy for Python applications, recommending the use of pytest as the primary testing framework.
It emphasizes the importance of writing unit tests, integration tests, and end-to-end tests to ensure code quality and reliability.
The ADR suggests using pytest fixtures for setting up test environments and using pytest plugins for extending testing capabilities.
It also recommends using coverage tools like pytest-cov to measure test coverage and ensure that critical code paths are adequately tested.
The ADR encourages writing tests for both new features and bug fixes, promoting a culture of testing and quality assurance.

### 0028 - Python Linting
This ADR discusses the use of linters for enforcing coding standards and style in Python applications.
It recommends using flake8 for static code analysis and enforcing PEP 8 coding standards, as well as using black for automatic code formatting.
The ADR emphasizes the importance of maintaining a consistent code style across the codebase and suggests using pre-commit hooks to automate linting checks before commits.

### 0029 - Python Packaging
This ADR outlines the standards for Python packaging, focusing on creating reliable and distributable Python packages.
It recommends using modern Python packaging tools like setuptools and build with pyproject.toml configuration.
The ADR emphasizes the importance of following semantic versioning for package releases and ensuring that packages have comprehensive documentation.

### 0030 - Python Dependency Management
This ADR discusses the approach to Python dependency management, recommending the use of uv for managing dependencies and virtual environments.
It emphasizes the importance of pinning dependencies with hash verification to ensure reproducible builds and prevent supply chain attacks.
The ADR also discusses strategies for handling transitive dependencies and resolving dependency conflicts.

### 0031 - Local Development Environment
This ADR outlines the standards for local development environments, recommending the use of dev containers or WSL for Windows development.
It emphasizes the importance of consistent development environments across the team to minimize "it works on my machine" issues.
The ADR recommends using uv for virtual environment management with `uv run` for executing commands rather than activating the virtual environment.
It also suggests using .env files for local environment variables with a .env-example file in the repository and .env in the gitignore file.

### 0032 - Web Framework Selection
This ADR discusses the criteria for selecting a web framework for Python applications, recommending Flask as the default choice for synchronous applications and FastAPI for applications requiring asynchronous operations or automatic OpenAPI documentation.
It outlines the trade-offs between the frameworks and provides guidance on when to use each one based on project requirements.
The ADR also discusses deployment considerations, recommending gunicorn for Flask applications and uvicorn for FastAPI applications in production.

### 0033 - HTTP Client Standards
This ADR outlines the standards for making HTTP requests in Python applications, recommending the use of requests for synchronous HTTP calls and httpx for applications requiring asynchronous HTTP calls or HTTP/2 support.
It emphasizes the importance of proper error handling, timeout configuration, and retry mechanisms when making HTTP requests.
The ADR also discusses best practices for working with RESTful APIs and handling various response formats.

### 0034 - Data Validation
This ADR establishes Pydantic as our standard tool for data validation, serialization, and settings management in Python applications. It details how to properly define models, use field validators, and leverage Pydantic's features for complex validation scenarios and settings management.

The ADR includes best practices for integrating Pydantic with our web frameworks (Flask and FastAPI), database models (SQLAlchemy), and API endpoints. It also covers performance considerations and testing strategies for validation logic.

### 0035 - Cloud Storage
This ADR outlines the approach to storing non-hot data, recommending the use of cloud object storage solutions like GCP Cloud Storage or AWS S3.
It emphasizes that databases should primarily be used for data that needs to be searched and indexed, while larger objects or infrequently accessed data should be stored in object storage.
The ADR discusses strategies for efficiently linking database records with objects in cloud storage and handling uploads and downloads securely.

### 0036 - Build Process
This ADR defines our containerized build process for Python applications. It establishes Docker as the standard containerization technology with multi-stage builds for optimized images. The ADR details best practices for Dockerfile creation, including security considerations, dependency management, and image optimization.

It also covers our CI/CD integration with Google Cloud Build, artifact management, and versioning strategy. The document includes guidelines for local development with docker-compose and production deployment with Cloud Run or Kubernetes.

### 0037 - Extranet
This ADR defines our extranet architecture for providing secure access to client services and sites during development and review phases. It establishes a pattern where client applications are deployed as containers behind an ingress controller with authentication enforcement.

The architecture uses cookie-based authentication with prefixed cookies (wwff_auth_) to avoid interference with the applications' own authentication mechanisms. The ADR details the implementation using Kubernetes ingress annotations, service deployment patterns, and domain configuration.

The document addresses access control management, local development integration, and the security considerations of this approach, ensuring a consistent and secure way to share work-in-progress services with clients and stakeholders.

### 0038 - Extranet Authentication
This ADR establishes our authentication system for the extranet infrastructure, based on OAuth 2.0 and OpenID Connect standards. It specifies Authelia as our identity provider, with support for federated authentication through external providers like Google and GitHub.

The ADR details the authentication flow, user management approach, and security considerations including cookie security, token management, and multi-factor authentication options. It provides configuration examples for both the Authelia service and its integration with external identity providers.

The document covers the deployment pattern as containerized applications in our Kubernetes cluster and addresses the consequences of this approach, including enhanced security and user convenience balanced against the complexity of configuration and management.

### 0039 - Helm
This ADR discusses the use of Helm for managing Kubernetes applications, recommending the use of Helm charts to package, configure, and deploy applications on Kubernetes.
It emphasizes the importance of using Helm for managing complex Kubernetes deployments, allowing for easy versioning and rollback of application releases.
The ADR also suggests using Helm templates for parameterizing application configurations, enabling customization of deployments based on different environments or use cases.
It also discusses best practices for organizing Helm charts, managing dependencies, and using Helm repositories for sharing charts within the organization or with the community.
It highlights the benefits of using Helm for simplifying Kubernetes application management and improving deployment consistency across environments.

### 0040 - FluxCD
This ADR discusses the use of FluxCD for continuous delivery in Kubernetes environments, recommending it as a GitOps solution for managing application deployments.
It emphasizes the importance of using Git as the single source of truth for application configurations and deployments, allowing for version control and auditability of changes.
The ADR outlines the key components of FluxCD, including the Flux controller, source controller, and kustomize controller, and how they work together to automate the deployment process.
We go a step further by using FluxCD to manage not only application deployments but also the entire Kubernetes cluster configuration, including networking, storage, and security policies through GitOps principles - this ensures that the entire cluster state is versioned and can be easily rolled back or audited.