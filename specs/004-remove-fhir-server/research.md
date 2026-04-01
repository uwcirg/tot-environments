# Research: Remove FHIR Server

**Feature Branch**: `004-remove-fhir-server`
**Date**: 2026-03-31

## Decision 1: FHIR Service Removal Scope

**Decision**: Remove both `fhir` (HAPI FHIR Server) and `fhir-auth` (JWT Proxy) services from all Docker Compose configurations (base, dev, prod).

**Rationale**: The `fhir-auth` service exists solely as an authentication proxy for the FHIR server. Removing one without the other would leave an orphaned service. Both must be removed together.

**Alternatives considered**:
- Remove only `fhir` and repurpose `fhir-auth` — rejected; the proxy has no purpose without an upstream FHIR server
- Disable services via profiles instead of removing — rejected; leaves configuration debt and confuses future maintainers

## Decision 2: cPRO FHIR Environment Variables

**Decision**: Remove `FHIR_R4_SERVER_ENDPOINT` and `FHIR_R4_EXTERNAL_ID_SYSTEM` environment variables from the cPRO service definition in `base/docker-compose.yaml` (lines 17-20).

**Rationale**: These variables configure cPRO to communicate with the FHIR server. With the server removed, these become dead references. The cPRO application should gracefully handle their absence (they act as optional feature switches).

**Alternatives considered**:
- Set variables to empty strings — rejected; cleaner to remove entirely, and the comment on line 19 confirms `FHIR_R4_EXTERNAL_ID_SYSTEM` acts as a feature switch (absence = feature disabled)

## Decision 3: Database Initialization Cleanup

**Decision**: Remove the `create database hapifhir;` line from `base/config/db/db.init.sql`.

**Rationale**: The `hapifhir` database is only used by the HAPI FHIR server. Creating it on new deployments wastes resources and misleads operators. The `db` (PostgreSQL) service itself must remain — it serves Keycloak.

**Alternatives considered**:
- Leave the database creation for backward compatibility — rejected; spec explicitly requires no FHIR databases on fresh deployments (FR-003)

## Decision 4: Keycloak SMART-on-FHIR Scope Removal

**Decision**: Remove the `patient/*.read` and `launch` client scope definitions and their references from `defaultOptionalClientScopes` in the Keycloak realm import JSON (`base/config/keycloak/import/tot-realm.json`).

**Rationale**: These scopes implement SMART-on-FHIR authorization (HL7 FHIR SMART App Launch). Without a FHIR server, they serve no purpose. Leaving them creates confusion about system capabilities. Located at:
- Scope definitions: lines 398-419 (two JSON objects)
- Default optional client scope references: lines 794-795

**Alternatives considered**:
- Keep scopes as harmless Keycloak configuration — rejected; spec requires clean removal (FR-004) and they mislead about system capabilities

## Decision 5: Environment File Cleanup

**Decision**: Delete `dev/fhir-auth.env.default` and `prod/fhir-auth.env.default`.

**Rationale**: These template files exist for the `fhir-auth` service's `env_file` directive. With the service removed, they are orphaned artifacts (FR-005).

**Alternatives considered**:
- None; straightforward deletion of service-specific config files

## Decision 6: PostgreSQL Service Retention

**Decision**: Retain the `db` (PostgreSQL) service and its health check configuration. Retain `db-data` volume.

**Rationale**: Keycloak depends on PostgreSQL (`KC_DB_URL_HOST: db`). The `db` service is shared infrastructure, not FHIR-specific. After FHIR removal, the init script will only create the `keycloak` database. The `db` service's `depends_on` relationships in dev/prod compose files reference it from both `keycloak` and `fhir` — the `fhir` dependency goes away with the service removal.

**Alternatives considered**:
- Remove PostgreSQL if only Keycloak uses it — rejected; Keycloak requires it, and the logserver also uses PostgreSQL
