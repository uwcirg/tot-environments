# Research: Remove SHL Components

**Feature**: 002-remove-shl-components | **Date**: 2026-03-31

## R1: Impact of removing SHL_MANAGER_URL from cpro

- **Decision**: Remove `SHL_MANAGER_URL` from cpro's environment in `base/docker-compose.yaml`
- **Rationale**: The cPRO application uses this variable to enable the SHL sharing feature. Without it, the feature is simply unavailable in the UI. The application does not fail without it — the spec confirms this assumption.
- **Alternatives considered**: Setting it to an empty string was considered but unnecessary; omitting the variable entirely is cleaner.

## R2: Impact of removing Keycloak SHL environment variables

- **Decision**: Remove `KEYCLOAK_SHL_CREATOR_BASE` and `KEYCLOAK_SHL_CREATOR_POST_LOGOUT_REDIRECT_URL` from keycloak's environment in `base/docker-compose.yaml`
- **Rationale**: These variables are used only for placeholder substitution in the `shl_creator` OIDC client definition within `tot-realm.json`. Since the `shl_creator` client is also being removed, these variables become unreferenced.
- **Alternatives considered**: Keeping variables without clients would cause Keycloak import warnings for unresolved placeholders — worse than removing both.

## R3: Impact of removing shl_creator OIDC client from tot-realm.json

- **Decision**: Remove the entire `shl_creator` client object from the `clients` array in `base/config/keycloak/import/tot-realm.json`
- **Rationale**: The OIDC client only serves the shl-creator service. Existing Keycloak instances that already imported this client will retain it (import strategy is `IGNORE_EXISTING`), but new deployments will not create it. This is safe.
- **Alternatives considered**: Disabling the client (setting `enabled: false`) was considered but leaves unnecessary configuration. Full removal is cleaner.

## R4: CORS TODO comment in fhir-auth

- **Decision**: Remove the TODO comment `# TODO review if necessary for shl-creator service in same deploy` from fhir-auth's labels in `base/docker-compose.yaml`
- **Rationale**: The TODO references shl-creator which no longer exists. The CORS middleware itself should remain as it serves the FHIR REST API regardless of shl-creator.
- **Alternatives considered**: Removing the entire CORS middleware was considered, but it may be needed by other FHIR clients. Only the TODO comment referencing shl-creator should be removed.

## R5: shl-server-data volume handling

- **Decision**: Remove the `shl-server-data` volume definition from dev and prod docker-compose files
- **Rationale**: The volume is only used by shl-server. After removing the service, the volume definition is orphaned. Existing Docker volumes on hosts will persist until manually removed by operators — this is documented in the spec's edge cases.
- **Alternatives considered**: None — orphaned volume definitions cause compose warnings.

## R6: shl-creator.env.default files

- **Decision**: Delete `dev/shl-creator.env.default` and `prod/shl-creator.env.default`
- **Rationale**: These template files exist per Principle VII (Sensitive Data Management) to document env vars for the shl-creator service. With the service removed, the templates are no longer needed.
- **Alternatives considered**: None — keeping them would violate the "no remnants" requirement (FR-005).
