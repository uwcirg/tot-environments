# Feature Specification: Remove FHIR Server

**Feature Branch**: `004-remove-fhir-server`
**Created**: 2026-03-31
**Status**: Draft
**Input**: User description: "This system does not need a FHIR server."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Remove FHIR Server from Deployment (Priority: P1)

An operations engineer deploys the TOT environment. After this change, the deployment no longer starts a HAPI FHIR server or its authentication proxy, reducing resource consumption and simplifying the running system.

**Why this priority**: The FHIR server and its auth proxy are unnecessary services consuming compute, memory, and operational attention. Removing them is the core purpose of this feature.

**Independent Test**: Deploy the environment and confirm that no FHIR-related containers are running, and that remaining services (cPRO, Keycloak, database) start and function correctly.

**Acceptance Scenarios**:

1. **Given** the updated Docker Compose configuration, **When** a developer runs `docker compose up`, **Then** no `fhir` or `fhir-auth` containers are started.
2. **Given** the updated deployment, **When** a developer inspects running containers, **Then** the system uses fewer resources compared to the previous configuration with FHIR services.

---

### User Story 2 - cPRO Application Functions Without FHIR (Priority: P1)

A clinical user accesses the cPRO (clinical patient-reported outcomes) application. The application loads and operates correctly without attempting to connect to a FHIR server endpoint.

**Why this priority**: If the cPRO application still references FHIR endpoints that no longer exist, it will fail. Ensuring the application works without FHIR is equally critical to removing the server itself.

**Independent Test**: Access the cPRO application through a browser, log in via Keycloak, and verify core functionality works without errors related to FHIR connectivity.

**Acceptance Scenarios**:

1. **Given** FHIR services are removed, **When** a user accesses the cPRO application, **Then** the application loads without FHIR-related errors.
2. **Given** FHIR services are removed, **When** the cPRO service starts, **Then** it does not attempt connections to a nonexistent FHIR endpoint.

---

### User Story 3 - Clean Up FHIR Database and Configuration Artifacts (Priority: P2)

An operations engineer reviews the deployment configuration. FHIR-specific artifacts (database initialization, Keycloak SMART-on-FHIR scopes, environment files) are removed, keeping the configuration clean and accurate.

**Why this priority**: While leftover artifacts don't actively cause failures, they waste resources, create confusion, and present a misleading picture of the system's capabilities.

**Independent Test**: Perform a fresh deployment and confirm no FHIR-related databases are created, no SMART-on-FHIR scopes exist in Keycloak, and no orphaned configuration files remain.

**Acceptance Scenarios**:

1. **Given** a fresh deployment, **When** the database initializes, **Then** no `hapifhir` database is created.
2. **Given** the updated Keycloak realm import, **When** the realm is loaded, **Then** no SMART-on-FHIR scopes are defined.
3. **Given** the updated configuration, **When** a developer inspects the repository, **Then** no FHIR-specific environment files remain (e.g., `fhir-auth.env.default`).

---

### Edge Cases

- What happens if the cPRO application has hardcoded references to FHIR endpoints beyond its environment variables?
- How does the system behave if an older deployment with FHIR data in the database is upgraded to this version?
- What happens if Traefik routing rules still reference removed FHIR service hostnames?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST NOT include `fhir` or `fhir-auth` service definitions in any Docker Compose configuration file (base, dev, prod).
- **FR-002**: System MUST NOT pass FHIR-related environment variables (`FHIR_R4_SERVER_ENDPOINT`, `FHIR_R4_EXTERNAL_ID_SYSTEM`) to the cPRO service.
- **FR-003**: System MUST NOT create the `hapifhir` database during database initialization.
- **FR-004**: System MUST NOT include SMART-on-FHIR scopes or FHIR-specific client configurations in the Keycloak realm import.
- **FR-005**: System MUST NOT include `fhir-auth.env.default` configuration files in dev or prod environments.
- **FR-006**: All remaining services (cPRO, Keycloak, database, reverse proxy) MUST continue to start and operate correctly after FHIR removal.
- **FR-007**: System MUST NOT define the `fhir-internal` network alias on any remaining service.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Zero FHIR-related containers running after a complete deployment.
- **SC-002**: All remaining services pass their health checks after FHIR removal.
- **SC-003**: Deployment starts 2 fewer containers (`fhir` and `fhir-auth`) compared to the current configuration.
- **SC-004**: No FHIR-related error messages appear in any service logs during startup or normal operation.

## Assumptions

- The cPRO application can function without a FHIR backend, or any FHIR-dependent features of cPRO are acceptable to lose.
- No other services beyond cPRO reference the FHIR server internally.
- Existing production data stored in the `hapifhir` database is not needed and does not require migration.
- The `fhir-internal` network alias is not used by any service other than `fhir-auth`.
- The FHIR image tag environment variable (`FHIR_IMAGE_TAG`) is not referenced elsewhere.
