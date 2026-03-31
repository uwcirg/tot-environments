# Feature Specification: Remove SHL Components

**Feature Branch**: `001-remove-shl-components`
**Created**: 2026-03-31
**Status**: Draft
**Input**: User description: "This application does not need the shl-* components (shl-creator, shl-server), and the other components do not need references to them."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Remove SHL Service Definitions (Priority: P1)

As a platform operator, I want the shl-creator and shl-server services completely removed from the Docker Compose configuration so that these unnecessary services are not deployed, reducing resource consumption and operational complexity.

**Why this priority**: These are the primary service definitions. Removing them is the core action and the foundation for all other cleanup.

**Independent Test**: Deploy the environment and verify that no shl-creator or shl-server containers are started, and all remaining services start and operate correctly.

**Acceptance Scenarios**:

1. **Given** the base docker-compose.yaml, **When** I run `docker compose config`, **Then** no `shl-creator` or `shl-server` service is listed.
2. **Given** the dev or prod docker-compose.yaml, **When** I run `docker compose config`, **Then** no `shl-creator` or `shl-server` service is listed.
3. **Given** the updated configuration, **When** I deploy the full stack, **Then** all remaining services (cpro, mysql, db, fhir, keycloak, fhir-auth) start successfully without errors.

---

### User Story 2 - Remove SHL References from Other Components (Priority: P1)

As a platform operator, I want all references to shl-creator and shl-server removed from other components' configurations (e.g., cPRO's `SHL_MANAGER_URL`, Keycloak's SHL-related variables and OIDC client) so that no component attempts to communicate with non-existent services.

**Why this priority**: Dangling references to removed services can cause startup errors, broken functionality, or confusing configuration. This is equally critical to removing the services themselves.

**Independent Test**: Search all configuration files for any `shl` references; confirm none remain. Deploy and verify no errors related to missing SHL endpoints appear in logs.

**Acceptance Scenarios**:

1. **Given** the cPRO service configuration, **When** I inspect its environment variables, **Then** no `SHL_MANAGER_URL` or other shl-related variables are present.
2. **Given** the Keycloak configuration, **When** I inspect its environment variables and realm import JSON, **Then** no `KEYCLOAK_SHL_CREATOR_BASE`, `KEYCLOAK_SHL_CREATOR_POST_LOGOUT_REDIRECT_URL`, or `shl_creator` OIDC client definitions are present.
3. **Given** the dev and prod environment directories, **When** I list all files, **Then** no `shl-creator.env.default` files exist.

---

### User Story 3 - Clean Up Volumes and Network References (Priority: P2)

As a platform operator, I want the `shl-server-data` volume definition and any SHL-only network associations removed so that the configuration is clean and does not reference unused resources.

**Why this priority**: While non-functional dangling volume definitions don't cause failures, they add confusion and clutter to the deployment configuration.

**Independent Test**: Run `docker compose config` and verify no `shl-server-data` volume is defined and no network references exist solely for SHL services.

**Acceptance Scenarios**:

1. **Given** the docker-compose configuration, **When** I inspect the volumes section, **Then** no `shl-server-data` volume is defined.
2. **Given** the docker-compose configuration, **When** I inspect network associations, **Then** no network membership exists solely to support SHL services.

---

### Edge Cases

- What happens if existing deployed environments have persistent `shl-server-data` volumes? Orphaned volumes remain on the host but do not affect new deployments; operators can manually remove them.
- What happens if external systems or documentation reference the `shl-creator` or `shl-server` URLs? Out of scope for this feature; only in-repo configuration is addressed.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The `shl-creator` service definition MUST be removed from the base docker-compose configuration.
- **FR-002**: The `shl-server` service definition MUST be removed from the base docker-compose configuration.
- **FR-003**: The `shl-creator` and `shl-server` service extensions MUST be removed from the dev docker-compose configuration.
- **FR-004**: The `shl-creator` and `shl-server` service extensions MUST be removed from the prod docker-compose configuration.
- **FR-005**: The `SHL_MANAGER_URL` environment variable MUST be removed from the cPRO service configuration.
- **FR-006**: The `KEYCLOAK_SHL_CREATOR_BASE` and `KEYCLOAK_SHL_CREATOR_POST_LOGOUT_REDIRECT_URL` environment variables MUST be removed from the Keycloak service configuration.
- **FR-007**: The `shl_creator` OIDC client definition MUST be removed from the Keycloak realm import JSON.
- **FR-008**: The `shl-creator.env.default` files MUST be removed from both dev and prod directories.
- **FR-009**: The `shl-server-data` volume definition MUST be removed from docker-compose configuration.
- **FR-010**: All remaining services MUST continue to function correctly after removal.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Zero references to `shl-creator`, `shl-server`, or `shl-` prefixed identifiers remain in any configuration file in the repository.
- **SC-002**: The full application stack deploys successfully with all remaining services healthy.
- **SC-003**: No errors or warnings related to missing SHL services appear in any service logs during deployment and operation.
- **SC-004**: The Keycloak authentication system operates correctly for all remaining clients after removal of the `shl_creator` client.

## Assumptions

- The SHL components (shl-creator, shl-server) are not in active use and can be safely removed without impact to end users.
- No external systems outside this repository depend on the SHL services being available at their configured URLs.
- The cPRO application functions correctly without the `SHL_MANAGER_URL` environment variable (i.e., the SHL sharing feature in cPRO is either unused or gracefully handles the absence of this configuration).
- Existing deployed `shl-server-data` Docker volumes on hosts will be left as orphans and can be manually cleaned up by operators.
- The Keycloak realm import is only used during initial setup; removing the `shl_creator` client from the import JSON does not affect already-provisioned Keycloak instances (manual cleanup of existing instances is out of scope).
