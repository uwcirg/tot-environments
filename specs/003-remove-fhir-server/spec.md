# Feature Specification: Remove FHIR Server

**Feature Branch**: `003-remove-fhir-server`
**Created**: 2026-03-31
**Status**: Draft
**Input**: User description: "This system does not need a FHIR server."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - System Deploys Without FHIR Services (Priority: P1)

A system operator deploying the TOT environment should be able to bring up the full stack without a FHIR server, FHIR auth proxy, or FHIR-related database. The deployment is simpler, uses fewer resources, and starts faster.

**Why this priority**: Removing unused infrastructure reduces operational complexity, resource consumption, and attack surface. This is the core deliverable of the feature.

**Independent Test**: Deploy the system using `docker compose up` and verify all remaining services start healthy without any FHIR-related containers running.

**Acceptance Scenarios**:

1. **Given** the updated Docker Compose configuration, **When** an operator runs `docker compose up`, **Then** no FHIR server, FHIR auth proxy, or FHIR database is created or started.
2. **Given** the updated configuration, **When** the system is fully running, **Then** all remaining services (cPRO, Keycloak, database, etc.) function correctly without errors related to missing FHIR endpoints.
3. **Given** the updated database initialization, **When** the PostgreSQL database starts, **Then** no `hapifhir` database is created.

---

### User Story 2 - cPRO Application Operates Without FHIR Dependencies (Priority: P1)

The cPRO application should function correctly without any FHIR server configuration. Environment variables and service references pointing to FHIR endpoints should be removed so the application does not attempt failed connections.

**Why this priority**: If cPRO still references FHIR endpoints that no longer exist, it could produce errors, degrade user experience, or fail to start. This is equally critical to removing the server itself.

**Independent Test**: Start the cPRO service and verify it does not log errors or warnings about unreachable FHIR endpoints, and that its core functionality remains intact.

**Acceptance Scenarios**:

1. **Given** the cPRO service with FHIR configuration removed, **When** cPRO starts, **Then** it does not attempt connections to any FHIR endpoint.
2. **Given** cPRO is running without FHIR, **When** a user accesses the application, **Then** all non-FHIR features work as expected.

---

### User Story 3 - Clean Environment Configuration (Priority: P2)

Environment-specific configurations (dev, prod) should not contain FHIR-related service definitions, environment files, or Traefik routing rules. The configuration should be clean and free of dead references.

**Why this priority**: Residual configuration for removed services creates confusion for operators and increases maintenance burden, but does not affect runtime behavior as critically as the other stories.

**Independent Test**: Review all environment-specific Docker Compose files and environment defaults to confirm no FHIR or fhir-auth references remain.

**Acceptance Scenarios**:

1. **Given** the dev environment configuration, **When** reviewed, **Then** no `fhir` or `fhir-auth` service definitions, Traefik rules, or environment files are present.
2. **Given** the prod environment configuration, **When** reviewed, **Then** no `fhir` or `fhir-auth` service definitions or environment files are present.

---

### Edge Cases

- What happens if cPRO receives a request that would normally trigger a FHIR interaction? The application should handle this gracefully (feature disabled or endpoint not available).
- What happens if Keycloak still has FHIR SMART App Launch scopes configured? These should be reviewed but may remain if they do not cause errors (scopes are informational).

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST NOT include the `fhir` service (HAPI FHIR server) in any Docker Compose configuration.
- **FR-002**: System MUST NOT include the `fhir-auth` service (JWT proxy for FHIR) in any Docker Compose configuration.
- **FR-003**: The database initialization script MUST NOT create the `hapifhir` database.
- **FR-004**: The cPRO service MUST NOT contain environment variables referencing FHIR endpoints (`FHIR_R4_SERVER_ENDPOINT`, `FHIR_R4_EXTERNAL_ID_SYSTEM`).
- **FR-005**: Environment-specific configurations (dev, prod) MUST NOT contain `fhir` or `fhir-auth` service extensions or environment files.
- **FR-006**: Traefik routing rules for FHIR endpoints MUST be removed from all configurations.
- **FR-007**: All remaining services MUST continue to start and pass health checks after FHIR removal.

### Key Entities

- **fhir service**: HAPI FHIR R4 server - to be removed from all Docker Compose configurations.
- **fhir-auth service**: JWT authentication proxy for FHIR - to be removed from all Docker Compose configurations.
- **hapifhir database**: PostgreSQL database used by HAPI FHIR - to be removed from database initialization.
- **cPRO service**: Patient-reported outcomes application - FHIR references to be removed from its configuration.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The system deploys successfully with zero FHIR-related containers running.
- **SC-002**: All remaining services reach healthy status after deployment.
- **SC-003**: No FHIR-related configuration (service definitions, environment variables, database initialization, routing rules) exists in any environment configuration file.
- **SC-004**: The cPRO application starts and serves requests without errors related to FHIR.
- **SC-005**: System resource usage (container count) decreases by at least 2 containers compared to the current deployment.

## Assumptions

- The cPRO application can operate without FHIR functionality, or its FHIR-dependent features are acceptable to lose.
- No other services in the stack beyond cPRO reference the FHIR server internally.
- Keycloak FHIR SMART App Launch scope definitions may remain in the realm configuration as they are informational and do not cause errors when the FHIR server is absent.
- The `fhir-auth.env.default` files in dev and prod environments should be removed along with the service definitions.
