---
parent: Decisions
nav_order: 38
title: 0038 Extranet Authentication
tags:
    - extranet
    - authentication
    - ingress
    - security
    - access-control
    - kubernetes
---
# Extranet Authentication

Date: 2025-06-20

## Status

Accepted

## Context

Our extranet infrastructure requires a robust, modern authentication system that supports:

1. **Federated Authentication**: Allow users to authenticate using established identity providers they already use
2. **Single Sign-On (SSO)**: Enable seamless access across multiple client services
3. **Fine-grained Authorization**: Control which users can access which services
4. **Security Best Practices**: Implement modern security standards
5. **User Management**: Easy administration of users and permissions

Traditional authentication methods (basic auth, custom login systems) would require substantial development and maintenance effort, while potentially lacking the security robustness of specialized authentication solutions.

## Decision

We will implement an extranet authentication system based on OAuth 2.0 and OpenID Connect (OIDC) principles, using Authelia or a similar identity provider (IdP) as the core authentication service. This service will integrate with external identity providers (Google, GitHub, etc.) for federated login while maintaining local user accounts for permission management.

### Authentication Architecture

```
┌───────────────────────────────────────────────────────┐
│                Extranet Authentication                │
│                                                       │
│  ┌─────────────┐      ┌───────────────────────────┐   │
│  │   Browser   │◄────►│     Ingress Controller    │   │
│  └─────────────┘      └───────────┬───────────────┘   │
│         │                         │                   │
│         │                         │                   │
│         │      ┌─────────────────┐│                   │
│         │      │     Auth        ││                   │
│         └─────►│   Middleware    │◄────────┐          │
│                └─────────────────┘         │          │
│                         │                  │          │
│                         ▼                  │          │
│                ┌──────────────────┐        │          │
│                │  Auth Service    │        │          │
│                │  (Authelia)      │        │          │
│                └────────┬─────────┘        │          │
│                         │                  │          │
│                         ▼                  │          │
│                ┌──────────────────┐        │          │
│                │  User Database   │───────►│          │
│                └────────┬─────────┘        │          │
│                         │                  │          │
│                         ▼                  │          │
│         ┌───────────────────────────────┐  │          │
│         │       External IdPs           │  │          │
│         │                               │  │          │
│         │  ┌───────────┐ ┌───────────┐  │  │          │
│         │  │  Google   │ │  GitHub   │  │──┘          │
│         │  └───────────┘ └───────────┘  │             │
│         │                               │             │
│         └───────────────────────────────┘             │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Authentication Service: Authelia

We will deploy [Authelia](https://www.authelia.com/) (or a similar solution) as our authentication service with the following features:

1. **Web Portal**: User-friendly login interface
2. **Multi-factor Authentication**: Support for TOTP (Time-based One-Time Passwords)
3. **Identity Validation**: Email verification for new accounts
4. **Federation**: Integration with external identity providers
5. **Authorization Rules**: Fine-grained access control policies

#### Authelia Configuration Example

```yaml
# authelia/configuration.yml
host: 0.0.0.0
port: 9091
log_level: info
jwt_secret: ${JWT_SECRET}
default_redirection_url: https://auth.wwff.tech

server:
  read_buffer_size: 4096
  write_buffer_size: 4096
  path: "/"

totp:
  issuer: wwff.tech
  period: 30
  skew: 1

authentication_backend:
  password_reset:
    disable: false
    custom_url: ""
  refresh_interval: 5m
  file:
    path: /config/users_database.yml

access_control:
  default_policy: deny
  rules:
    - domain: "*.extranet.wwff.tech"
      policy: bypass
      subject: "group:admins"
    - domain: "client-x.extranet.wwff.tech"
      policy: one_factor
      subject: "group:client-x"
    - domain: "client-y.extranet.wwff.tech"
      policy: one_factor
      subject: "group:client-y"

session:
  name: wwff_auth_session
  domain: wwff.tech
  same_site: lax
  expiration: 12h
  inactivity: 30m
  remember_me_duration: 30d

regulation:
  max_retries: 5
  find_time: 2m
  ban_time: 5m

storage:
  postgres:
    host: postgres
    port: 5432
    database: authelia
    username: authelia
    password: ${DB_PASSWORD}
    
notifier:
  smtp:
    host: smtp.gmail.com
    port: 587
    username: ${SMTP_USERNAME}
    password: ${SMTP_PASSWORD}
    sender: auth@wwff.tech
```

### Federation with External Identity Providers

We will implement OpenID Connect (OIDC) integration with the following providers:

#### 1. Google

```yaml
# authelia/configuration.yml (partial)
identity_providers:
  oidc:
    hmac_secret: ${OIDC_HMAC_SECRET}
    issuer_private_key: ${OIDC_PRIVATE_KEY}
    cors:
      allowed_origins:
        - https://*.extranet.wwff.tech
    clients:
      - id: google
        description: Google
        secret: ${GOOGLE_CLIENT_SECRET}
        authorization_policy: two_factor
        redirect_uris:
          - https://auth.wwff.tech/oidc/callback
        scopes:
          - openid
          - profile
          - email
```

#### 2. GitHub

```yaml
# authelia/configuration.yml (partial)
identity_providers:
  oidc:
    # ... other configurations
    clients:
      # ... other clients
      - id: github
        description: GitHub
        secret: ${GITHUB_CLIENT_SECRET}
        authorization_policy: two_factor
        redirect_uris:
          - https://auth.wwff.tech/oidc/callback
        scopes:
          - openid
          - profile
          - email
```

### User Management

User management will include:

1. **Local User Database**: Store user accounts, groups, and permissions
2. **User Profile Mapping**: Map external IdP profiles to local accounts
3. **Group Membership**: Organize users into groups for easier permission management
4. **Admin Interface**: Web interface for user and permission administration

#### User Database Structure

```yaml
# users_database.yml example
users:
  john:
    displayname: "John Doe"
    password: "$argon2id$v=19$m=65536,t=3,p=4$..."
    email: john.doe@example.com
    groups:
      - admins
      - client-x
    
  alice:
    displayname: "Alice Smith"
    password: "$argon2id$v=19$m=65536,t=3,p=4$..."
    email: alice.smith@example.com
    groups:
      - client-y
      - developers
```

### Authentication Flow

1. **Initial Access**:
   - User attempts to access a protected service (`client-x.extranet.wwff.tech`)
   - Ingress controller redirects to authentication service

2. **Authentication**:
   - User sees login screen with options for federated providers (Google, GitHub)
   - User selects preferred provider and authenticates
   
3. **Account Linking**:
   - For first-time users, a local account is created and linked to the external identity
   - For returning users, the system identifies their existing local account

4. **Authorization Check**:
   - System verifies user has permission to access the requested service
   - If authorized, authentication cookie is set and user is redirected to the service

5. **Session Management**:
   - Authentication cookie maintains the session across the extranet
   - Session expires after inactivity or maximum duration
   - "Remember me" option available for longer sessions

### Security Considerations

1. **Cookie Security**:
   - Prefixed with `wwff_auth_` to avoid conflicts
   - `Secure` flag to ensure HTTPS-only
   - `HttpOnly` flag to prevent JavaScript access
   - `SameSite=Lax` to mitigate CSRF

2. **Token Security**:
   - Short-lived JWT tokens
   - Regular key rotation
   - Token validation on each request

3. **Password Security** (for direct login):
   - Argon2id hashing algorithm
   - Password complexity requirements
   - Brute force protection

4. **Multi-factor Authentication**:
   - TOTP-based second factor
   - Optional or required based on policy

### Deployment

The authentication service will be deployed as a containerized application in our Kubernetes cluster:

```yaml
# kubernetes/authelia-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: authelia
  namespace: auth
spec:
  replicas: 2
  selector:
    matchLabels:
      app: authelia
  template:
    metadata:
      labels:
        app: authelia
    spec:
      containers:
      - name: authelia
        image: authelia/authelia:latest
        ports:
        - containerPort: 9091
        volumeMounts:
        - name: config
          mountPath: /config
        env:
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: authelia-secrets
              key: jwt-secret
        # Additional environment variables
      volumes:
      - name: config
        configMap:
          name: authelia-config
```

## Consequences

### Positive

- **Enhanced Security**: Leverages established identity providers with strong security practices
- **User Convenience**: Allows users to authenticate with accounts they already have
- **Reduced Development Effort**: Uses existing, well-tested authentication solutions
- **Centralized Management**: Single point for managing user access across all client services
- **Standards Compliance**: Follows OAuth 2.0 and OpenID Connect standards

### Negative

- **Complexity**: More complex than simple authentication methods
- **External Dependencies**: Relies on third-party identity providers
- **Configuration Overhead**: Requires careful configuration for each client service
- **Migration Challenges**: May require user re-enrollment if changing authentication providers
- **Potential Privacy Concerns**: Collects user information from external identity providers

## Related Decisions

* [0037 Extranet](./0037-extranet.md)
* [0001 IAM](./0001-IAM.md)
* [0009 Application Security](./0009-application-security.md)
* [0020 Kubernetes](./0020-kubernetes.md)
