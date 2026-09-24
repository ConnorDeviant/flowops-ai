
# FlowOps AI — System Architecture

**Version:** 1.0
**Status:** Approved Blueprint / Implementation Pending
**Primary Demo:** Digital Marketing Agency

## 1. Overview

FlowOps AI is an AI-powered, multi-tenant B2B Sales and
Business Operations SaaS.

The platform integrates CRM, project management, billing,
workflow automation, AI assistance and analytics.

The initial implementation focuses on digital marketing
agency operations.

## 2. Technology Stack

| Layer | Technology |
|---|---|
| Backend | Java 21, Spring Boot |
| Architecture | Microservices, Spring Cloud Gateway |
| Security | Spring Security, JWT |
| Database | PostgreSQL, Flyway |
| Messaging | Apache Kafka |
| Caching | Redis |
| Frontend | React, TypeScript, Vite |
| UI | Tailwind CSS, shadcn/ui |
| Testing | JUnit 5, Mockito, Testcontainers |
| DevOps | Docker, Docker Compose, GitHub Actions |

## 3. Microservice Boundaries

### API Gateway

Responsibilities:
- Route requests to backend services.
- Validate JWT access tokens.
- Configure CORS.
- Apply rate limiting where required.

The Gateway is not the sole authorization boundary.
Backend services must independently enforce security.

### Identity Service

Responsibilities:
- Organization registration.
- User management.
- Role and membership management.
- Password hashing and verification.
- JWT authentication.
- Tenant context management.

Owns the Identity database.

### CRM Service

Responsibilities:
- Lead management.
- Customer management.
- Sales pipelines.
- Lead activities and follow-ups.

Owns the CRM database.

### Operations Service

Responsibilities:
- Projects and tasks.
- Milestones.
- Invoicing and payment tracking.
- Operational reporting.

Owns the Operations database.

### Automation & AI Service

Responsibilities:
- Kafka event consumption.
- Workflow execution.
- AI assistance.
- Notifications.
- Workflow execution history.

Owns the Automation database.

## 4. Database Ownership

Each service owns its database.

Services must not directly access another service's
database tables.

Cross-service references use UUIDs.

Inter-service communication uses APIs or versioned events.

The initial deployment uses shared PostgreSQL infrastructure
with separate service-owned databases.

## 5. Multi-Tenancy

The MVP uses shared database infrastructure with
tenant-scoped business records.

Security requirements:

1. Business records contain tenant_id.
2. Tenant identity comes from authenticated JWT claims.
3. Client-provided tenant IDs cannot override JWT identity.
4. Services enforce role-based authorization.
5. Repository queries must enforce tenant isolation.
6. Sensitive operations require audit logging.
7. Cross-tenant access attempts must be rejected.

Initial organization roles:
- ORG_ADMIN
- MANAGER
- EMPLOYEE

## 6. Authentication

Day 1 authentication flow:

1. A company submits its registration request.
2. Identity Service validates the request.
3. An organization and first administrator are created.
4. The administrator's password is securely hashed.
5. The administrator submits login credentials.
6. Identity Service authenticates the administrator.
7. A signed JWT access token is issued.
8. The client sends the JWT with protected requests.
9. The Gateway validates the JWT.
10. The destination service independently enforces
    authentication, authorization and tenant isolation.

JWT signing keys must never be committed to Git.

## 7. Initial Identity APIs

POST /api/v1/auth/register
POST /api/v1/auth/login
GET  /api/v1/users/me

## 8. Day 1 Scope

Implement:
- Identity Service.
- API Gateway.
- PostgreSQL and Flyway.
- Organization registration.
- Administrator creation.
- JWT authentication.
- Role-based authorization.
- Tenant isolation.
- Docker Compose development environment.
- Automated authentication and isolation tests.

## 9. Deferred Implementation

Day 2: CRM backend.

Day 3: React frontend.

Day 4: Operations and billing.

Day 5: Kafka automation and AI.

Day 6: Analytics and integration testing.

Day 7: Deployment and portfolio documentation.

Future industry extensions remain outside Day 1 scope.

## 10. Architecture Decisions

ADR-001: Use four business microservices.

ADR-002: Use a monorepo.

ADR-003: Use separate service-owned databases.

ADR-004: Derive tenant context from authenticated JWTs.

ADR-005: Enforce authorization inside backend services.

ADR-006: Use versioned Flyway database migrations.

Exact Spring Boot and Spring Cloud versions will be
selected after compatibility verification.

JWT signing implementation and membership constraints
will be finalized in the Identity Service design.