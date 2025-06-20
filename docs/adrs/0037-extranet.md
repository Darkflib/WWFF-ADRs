---
parent: Decisions
nav_order: 37
title: 0037 Extranet
tags:
    - extranet
    - authentication
    - ingress
    - security
    - access-control
    - kubernetes
---
# Extranet

Date: 2025-06-20

## Status

Accepted

## Context

When developing services and sites for clients, we often need to provide a secure way for authorized personnel to access these services during development, testing, and client review phases. Traditional approaches like VPNs or basic authentication have limitations in terms of user experience, management overhead, and integration with modern authentication systems.

We need a solution that provides:

1. **Centralized Access Control**: A single point for managing access to multiple client projects
2. **Non-intrusive Authentication**: Authentication layer that doesn't interfere with the application's own authentication mechanisms
3. **Fine-grained Permissions**: Ability to control which users can access which services
4. **Modern Security Standards**: Support for current security best practices
5. **Ease of Deployment**: Simple integration with our containerized application architecture

## Decision

We will implement an extranet architecture that places client services and sites behind a unified authentication layer using an ingress controller with authentication enforcement. This system will operate separately from any internal authentication mechanisms within the deployed applications themselves.

### Architecture Overview

The extranet will consist of these primary components:

1. **Ingress Controller with Auth Middleware**: Intercepts all requests to protected services and verifies authentication
2. **Identity Provider (IdP)**: Handles authentication and issues tokens/cookies (detailed in [0038 Extranet Authentication](./0038-extranet-authentication.md))
3. **Protected Services**: Client applications and sites deployed as containers
4. **Permission Management System**: Database and admin interface for managing access control

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │  Service A  │    │  Service B  │    │  Service C  │  │
│  │ (Client X)  │    │ (Client Y)  │    │ (Client Z)  │  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
│          ▲                 ▲                 ▲          │
│          │                 │                 │          │
│          │                 │                 │          │
│  ┌───────┴─────────────────┴─────────────────┴────────┐ │
│  │                 Ingress Controller                 │◄┼─┐
│  │          (with Auth Middleware/Enforcer)           │ │ │
│  └───────────────────────┬────────────────────────────┘ │ │
│                          │                              │ │
│                          │ Auth Request                 │ │
│                          ▼                              │ │
│  ┌──────────────────────────────────────────┐           │ │
│  │           Identity Provider              │           │ │
│  │         (Authelia or similar)            │           │ │
│  └──────────────────────────────────────────┘           │ │
│                                                         │ │
└─────────────────────────────────────────────────────────┘ │
                            ▲                               │
                            │                               │
                            │ Redirect if not               │
                            │ authenticated                 │
                            │                               │
                   ┌────────┴──────────┐                    │
                   │      Browser      │◄───────────────────┘
                   └─────────┬─────────┘
                             │
                             │
                   ┌─────────▼─────────┐
                   │       User        │
                   └───────────────────┘
```

### Implementation Details

#### 1. Ingress Controller Configuration

We will use Kubernetes ingress with annotations for authentication enforcement:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: client-service-ingress
  annotations:
    # Authentication related annotations
    nginx.ingress.kubernetes.io/auth-url: "https://auth.wwff.tech/api/verify"
    nginx.ingress.kubernetes.io/auth-signin: "https://auth.wwff.tech/login"
    nginx.ingress.kubernetes.io/auth-response-headers: "Remote-User,Remote-Name,Remote-Email,Remote-Groups"
spec:
  rules:
  - host: client-x.extranet.wwff.tech
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: client-x-service
            port:
              number: 80
```

#### 2. Cookie-Based Authentication

To prevent interference with application-level authentication:

- Authentication cookies will use a specific prefix (e.g., `wwff_auth_`)
- Cookies will be set with appropriate security attributes:
  - `Secure`: Only sent over HTTPS
  - `HttpOnly`: Not accessible via JavaScript
  - `SameSite=Lax`: Mitigate CSRF attacks while maintaining usability

#### 3. Authentication Flow

1. User attempts to access a protected service (`client-x.extranet.wwff.tech`)
2. Ingress controller checks for valid authentication cookies
3. If not authenticated, user is redirected to the authentication service
4. After successful authentication, user is redirected back to the original service
5. The authentication cookie is verified by the ingress controller
6. If valid, the request is forwarded to the actual service

#### 4. Service Deployment

Each client service will be deployed as one or more containers, typically using:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-x-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: client-x
  template:
    metadata:
      labels:
        app: client-x
    spec:
      containers:
      - name: client-x-app
        image: gcr.io/wwff-projects/client-x:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: client-x-service
spec:
  selector:
    app: client-x
  ports:
  - port: 80
    targetPort: 80
```

#### 5. Domain Configuration

- Each client project will have a subdomain under `extranet.wwff.tech`
- DNS entries will be automatically managed via our CI/CD pipeline
- TLS certificates will be automatically provisioned and renewed using cert-manager or similar tools

### Access Control Management

The system will provide:

1. **User Management**: Create, update, and delete user accounts
2. **Group Management**: Organize users into groups (e.g., by client, project, or role)
3. **Permission Assignment**: Grant access to specific services based on user or group
4. **Access Logs**: Record all authentication events and access attempts

### Local Development Integration

For local development, developers can:

1. Use a local version of the authentication service for testing
2. Simulate the extranet environment using Docker Compose:

```yaml
# docker-compose.yml
version: '3'
services:
  auth-service:
    image: authelia/authelia:latest
    volumes:
      - ./authelia:/config
    ports:
      - "9091:9091"
    environment:
      - TZ=Europe/London
      
  nginx-proxy:
    image: nginx:latest
    volumes:
      - ./nginx:/etc/nginx/conf.d
    ports:
      - "80:80"
    depends_on:
      - auth-service
      - client-service
      
  client-service:
    build: .
    ports:
      - "8080:8080"
```

## Consequences

### Positive

- **Unified Authentication**: Single sign-on experience across all client projects
- **Separation of Concerns**: Authentication layer independent from application logic
- **Enhanced Security**: Modern authentication standards with federated identity
- **Simplified Access Management**: Centralized permission control
- **Non-intrusive**: Client applications function normally with their own authentication if present

### Negative

- **Additional Infrastructure**: Requires maintaining authentication infrastructure
- **Configuration Overhead**: Each service needs proper ingress configuration
- **Potential Single Point of Failure**: Authentication service availability affects all client services
- **Cookie Management Complexity**: Need to carefully manage cookie namespaces and attributes

## Related Decisions

* [0038 Extranet Authentication](./0038-extranet-authentication.md)
* [0020 Kubernetes](./0020-kubernetes.md)
* [0001 IAM](./0001-IAM.md)
* [0007 Application Deployment](./0007-application-deployment.md)
* [0009 Application Security](./0009-application-security.md)
