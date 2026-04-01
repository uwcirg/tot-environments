# Quickstart: Remove FHIR Server

**Feature Branch**: `004-remove-fhir-server`

## Implementation Order

### Step 1: Remove FHIR services from base Docker Compose

**File**: `base/docker-compose.yaml`
- Delete the `fhir` service block (lines 81-113)
- Delete the `fhir-auth` service block (lines 159-184)
- Remove `FHIR_R4_SERVER_ENDPOINT` and `FHIR_R4_EXTERNAL_ID_SYSTEM` from cPRO environment (lines 17-20, including comments)

### Step 2: Remove FHIR services from dev Docker Compose

**File**: `dev/docker-compose.yaml`
- Delete the `fhir` service block (lines 21-34)
- Delete the `fhir-auth` service block (lines 45-50)

### Step 3: Remove FHIR services from prod Docker Compose

**File**: `prod/docker-compose.yaml`
- Delete the `fhir` service block (lines 21-26)
- Delete the `fhir-auth` service block (lines 37-42)

### Step 4: Clean up database initialization

**File**: `base/config/db/db.init.sql`
- Remove the `---- HAPI ----` comment and `create database hapifhir;` line (lines 5-6)

### Step 5: Clean up Keycloak realm import

**File**: `base/config/keycloak/import/tot-realm.json`
- Remove `patient/*.read` scope definition (lines 398-407)
- Remove `launch` scope definition (lines 409-419)
- Remove `"launch",` and `"patient/*.read",` from `defaultOptionalClientScopes` array (lines 794-795)

### Step 6: Delete FHIR environment files

- Delete `dev/fhir-auth.env.default`
- Delete `prod/fhir-auth.env.default`

## Verification

1. Run `docker compose -f dev/docker-compose.yaml config` — should produce valid YAML with no FHIR references
2. Run `docker compose -f prod/docker-compose.yaml config` — same check
3. Grep for `fhir` across all YAML and config files — should return zero results (except specs/)
4. Verify remaining services in compose config: cpro, mysql, db, keycloak (+ traefik from external)

## Risks

- **cPRO hardcoded FHIR references**: The cPRO application image may contain code that references FHIR. Since cPRO is a pulled image (not built here), this is outside scope — the spec assumes cPRO functions without FHIR.
- **Existing deployments**: Running systems with `hapifhir` data in PostgreSQL are unaffected; the init script only runs on fresh databases.
