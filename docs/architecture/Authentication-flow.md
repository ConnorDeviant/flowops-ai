
# FlowOps AI — Authentication & Tenant Isolation

**Version:** 1.0
**Status:** Proposed Day 1 implementation
**Owner:** Identity Service

## 1. Objective

Implement secure JWT-based authentication and
tenant-aware authorization for FlowOps AI.

The API Gateway validates incoming JWTs, while
each backend service independently enforces
authentication, authorization and tenant isolation.

## 2. Company Registration

Endpoint:
POST /api/v1/auth/register

Example request:

{
"organizationName": "Acme Digital Agency",
"organizationSlug": "acme-digital",
"adminName": "Agency Administrator",
"adminEmail": "admin@example.com",
"password": "<user-provided-password>"
}

Registration flow:

1. Validate the request.
2. Normalize the administrator email.
3. Check organization slug and email uniqueness.
4. Generate server-side UUIDs.
5. Hash the administrator password.
6. Create the organization.
7. Create the administrator account.
8. Assign the ORG_ADMIN membership.
9. Commit the transaction.
10. Return organization and administrator identifiers.

All database writes must occur within one transaction.

Never return password hashes.

## 3. Login

Endpoint:
POST /api/v1/auth/login

Example request:

{
"email": "admin@example.com",
"password": "<user-provided-password>",
"organizationSlug": "acme-digital"
}

Login flow:

1. Validate credentials.
2. Load the user by normalized email.
3. Verify the password using Spring Security.
4. Resolve the requested organization.
5. Verify the user's active membership.
6. Retrieve the membership role.
7. Generate a signed tenant-scoped JWT.
8. Return the access token.

Invalid credentials or unauthorized organization
membership must not result in token issuance.

## 4. JWT Design

Proposed access-token claims:

- sub: authenticated user UUID
- tenant_id: active organization UUID
- roles: authorized organization roles
- iss: configured token issuer
- aud: intended API audience
- iat: token issuance time
- exp: token expiration time
- jti: unique token identifier

JWT signing keys must be loaded from secure
environment-based configuration.

Never store credentials or signing keys in Git.

Token lifetime, signing algorithm and key-rotation
policy must be finalized before implementation.

## 5. JWT Validation

For protected requests:

1. Read the Authorization: Bearer token.
2. Verify the JWT signature.
3. Validate issuer and audience.
4. Validate token expiration.
5. Extract the authenticated user and tenant.
6. Extract the authorized roles.
7. Create the authenticated security context.
8. Forward the request to the destination service.

Backend services must independently validate
authentication and enforce authorization.

## 6. Tenant Resolution

Tenant identity comes exclusively from the
validated JWT.

A client-provided tenant_id in request bodies,
query parameters or headers must never override
the authenticated tenant.

All tenant-owned business records must be
accessed using tenant-scoped repository queries.

A resource identifier alone is insufficient
authorization to access that resource.

## 7. Role-Based Authorization

Initial organization roles:

- ORG_ADMIN
- MANAGER
- EMPLOYEE

Day 1 authorization:

- Public users may register and log in.
- Authenticated users may access their own profile.
- Organization administration requires ORG_ADMIN.
- Tenant-owned resources require matching tenant
  identity and appropriate role permissions.

Additional endpoint permissions will be documented
when their corresponding services are implemented.

## 8. API Gateway Responsibilities

The Gateway handles:

- Request routing.
- JWT verification for protected routes.
- CORS configuration.
- Rejection of invalid or expired tokens.
- Public authentication route configuration.

The Gateway must not be the only security boundary.

Services must not trust arbitrary identity headers
provided by external clients.

## 9. Error Handling

Expected HTTP responses:

400 — Invalid registration or login request.
401 — Missing, invalid or expired authentication.
403 — Authenticated user lacks required permission.
409 — Organization slug or user email conflict.

Cross-tenant resource requests may return 404
to avoid disclosing another tenant's resource.

Never expose password hashes, JWT signing keys,
stack traces or sensitive authentication details
in API responses or logs.

## 10. Required Security Tests

- Successful company registration.
- Administrator role assignment.
- Registration transaction rollback.
- Duplicate registration rejection.
- Successful administrator login.
- Invalid password rejection.
- Invalid or expired JWT rejection.
- Protected endpoint authentication.
- Unauthorized role rejection.
- Client-supplied tenant override rejection.
- Cross-tenant resource access rejection.

## 11. Implementation Status

Architecture documentation: Proposed.
Registration API: Pending.
Login API: Pending.
JWT implementation: Pending.
Gateway security: Pending.
Tenant-isolation tests: Pending.