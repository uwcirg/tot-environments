# Feature Specification: Remove SHL Components

**Feature Branch**: `002-remove-shl-components`
**Created**: 2026-03-31
**Status**: Draft
**Input**: User description: "This application does not need the shl-* components (shl-creator, shl-server), and the other components do not need references to them."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Remove SHL services from deployment (Priority: P1)

As a platform operator, I want the shl-creator and shl-server services completely removed from the Docker Compose configuration so that the deployment only includes the services actually needed by this application.

**Why this priority**: The SHL services are unnecessary for this deployment. Removing them simplifies the environment, reduces resource consumption, and eliminates maintenance burden for unused components.

**Independent Test**: Deploy the environment without shl-creator and shl-server services and verify all remaining services (cpro, mysql, db, fhir, keycloak, fhir-auth) start and function correctly.

**Acceptance Scenarios**:

1. **Given** the base docker-compose configuration, **When** a platform operator deploys the environment, **Then** no shl-creator or shl-server containers are created or started
2. **Given** the dev docker-compose configuration, **When** a platform operator deploys the dev environment, **Then** no shl-creator or shl-server service definitions or env_file references exist
3. **Given** the prod docker-compose configuration, **When** a platform operator deploys the prod environment, **Then** no shl-creator or shl-server service definitions or env_file references exist

---

### User Story 2 - Remove SHL references from remaining services (Priority: P1)

As a platform operator, I want all references to shl-creator and shl-server removed from the remaining services' configurations (cpro, keycloak, fhir-auth) so that no component depends on or points to the removed SHL services.

**Why this priority**: Leaving stale references to removed services creates confusion, potential startup warnings, and misleading configuration. This is equally critical as removing the services themselves.

**Independent Test**: After removing SHL references, verify that cpro, keycloak, and fhir-auth start without errors and do not contain any environment variables or configuration entries pointing to shl-creator or shl-server.

**Acceptance Scenarios**:

1. **Given** the cpro service configuration, **When** reviewing its environment variables, **Then** no `SHL_MANAGER_URL` or other shl-related variables are present
2. **Given** the keycloak service configuration, **When** reviewing its environment variables, **Then** no `KEYCLOAK_SHL_CREATOR_BASE` or `KEYCLOAK_SHL_CREATOR_POST_LOGOUT_REDIRECT_URL` variables are present
3. **Given** the keycloak realm import configuration, **When** reviewing the tot-realm.json, **Then** no `shl_creator` OIDC client definition exists

---

### User Story 3 - Clean up SHL-related artifacts (Priority: P2)

As a platform operator, I want all SHL-related support files (env defaults, volumes, image tag variables) removed so that the project contains no remnants of the SHL components.

**Why this priority**: Residual files and configuration create confusion for future maintainers. This is important for long-term clarity but lower priority than functional removal.

**Independent Test**: Search the entire project for any remaining references to "shl" and confirm none exist in active configuration files.

**Acceptance Scenarios**:

1. **Given** the dev and prod directories, **When** checking for env files, **Then** no `shl-creator.env.default` files exist
2. **Given** the base docker-compose configuration, **When** reviewing volume definitions, **Then** no `shl-server-data` volume is defined
3. **Given** the environment configuration, **When** reviewing image tag variables, **Then** no `SHL_CREATOR_IMAGE_TAG` or `SHL_SERVER_IMAGE_TAG` variables are referenced

### Edge Cases

- What happens if an existing deployment has a `shl-server-data` Docker volume with data? The volume will remain on the host but will no longer be managed by compose; operators may need to manually remove it.
- What happens if external systems or bookmarks reference `shl-creator.${BASE_DOMAIN}` or `shl-server.${BASE_DOMAIN}` URLs? Those URLs will no longer resolve; this is expected behavior since the services are intentionally removed.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST NOT define shl-creator or shl-server services in any docker-compose configuration file (base, dev, prod)
- **FR-002**: System MUST NOT contain environment variables referencing shl-creator or shl-server in any remaining service configuration (cpro, keycloak, fhir-auth)
- **FR-003**: System MUST NOT define the `shl-server-data` volume in any docker-compose configuration
- **FR-004**: System MUST NOT include the `shl_creator` OIDC client in the keycloak realm import configuration
- **FR-005**: System MUST NOT include shl-creator.env.default files in dev or prod directories
- **FR-006**: All remaining services (cpro, mysql, db, fhir, keycloak, fhir-auth) MUST continue to function correctly after SHL component removal
- **FR-007**: System MUST NOT reference `SHL_CREATOR_IMAGE_TAG` or `SHL_SERVER_IMAGE_TAG` variables in any configuration

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Zero references to "shl-creator" or "shl-server" exist in any active configuration file across the project
- **SC-002**: All remaining services start successfully without errors related to missing SHL components
- **SC-003**: The deployment footprint is reduced by exactly two services (shl-creator and shl-server) and one persistent volume (shl-server-data)
- **SC-004**: Platform operators can deploy the full environment without needing any SHL-related configuration or environment files

## Assumptions

- Existing deployments may have `shl-server-data` Docker volumes that will need manual cleanup outside of this change
- No other external systems critically depend on the SHL services being available at this deployment
- The cpro application will function correctly without the `SHL_MANAGER_URL` feature (the SHL sharing functionality will simply be unavailable)
- The keycloak realm will function correctly without the `shl_creator` OIDC client definition
- Any TODO comments referencing SHL services (e.g., CORS configuration notes for shl-creator in fhir-auth) should also be removed
