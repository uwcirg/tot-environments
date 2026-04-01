# Data Model: Remove FHIR Server

**Feature Branch**: `004-remove-fhir-server`
**Date**: 2026-03-31

## Configuration Objects Being Removed

This feature removes infrastructure configuration rather than application data models.
The following configuration objects are inventoried for removal.

### Docker Compose Services

| Object | Location | Type | Dependencies |
|--------|----------|------|--------------|
| `fhir` service | `base/docker-compose.yaml:81-113` | Service definition | `db` (PostgreSQL), `fhir-internal` network alias |
| `fhir` service (dev) | `dev/docker-compose.yaml:21-34` | Service extension | Extends base `fhir`, adds ingress network + Traefik labels |
| `fhir` service (prod) | `prod/docker-compose.yaml:21-26` | Service extension | Extends base `fhir`, adds `db` dependency |
| `fhir-auth` service | `base/docker-compose.yaml:159-184` | Service definition | `fhir-internal:8080` upstream, Keycloak OIDC endpoints |
| `fhir-auth` service (dev) | `dev/docker-compose.yaml:45-50` | Service extension | Extends base `fhir-auth`, depends on `fhir` |
| `fhir-auth` service (prod) | `prod/docker-compose.yaml:37-42` | Service extension | Extends base `fhir-auth`, depends on `fhir` |

### Environment Variables

| Variable | Location | Purpose |
|----------|----------|---------|
| `FHIR_R4_SERVER_ENDPOINT` | `base/docker-compose.yaml:18` | cPRO FHIR server URL |
| `FHIR_R4_EXTERNAL_ID_SYSTEM` | `base/docker-compose.yaml:20` | cPRO FHIR identifier system (feature switch) |
| `FHIR_IMAGE_TAG` | `base/docker-compose.yaml:82` | HAPI FHIR image version (removed with service) |
| `PROXY_IMAGE_TAG` | `base/docker-compose.yaml:160` | JWT proxy image version (removed with service) |

### Environment Files

| File | Purpose |
|------|---------|
| `dev/fhir-auth.env.default` | Template for fhir-auth service secrets (dev) |
| `prod/fhir-auth.env.default` | Template for fhir-auth service secrets (prod) |

### Database

| Object | Location | Purpose |
|--------|----------|---------|
| `hapifhir` database | `base/config/db/db.init.sql:6` | PostgreSQL database for HAPI FHIR data |

### Keycloak Configuration

| Object | Location | Purpose |
|--------|----------|---------|
| `patient/*.read` scope | `tot-realm.json:398-407` | SMART-on-FHIR read scope |
| `launch` scope | `tot-realm.json:409-419` | SMART-on-FHIR launch context |
| `launch` default ref | `tot-realm.json:794` | Default optional client scope |
| `patient/*.read` default ref | `tot-realm.json:795` | Default optional client scope |

### Network Configuration

| Object | Location | Purpose |
|--------|----------|---------|
| `fhir-internal` alias | `base/docker-compose.yaml:110-111` | Internal network alias (removed with `fhir` service) |

## Objects Retained

| Object | Reason |
|--------|--------|
| `db` (PostgreSQL) service | Required by Keycloak |
| `db-data` volume | Stores Keycloak data |
| `db.init.sql` file | Still needed for `keycloak` database creation |
| `internal` network | Used by remaining services |
| `ingress` network | Used by remaining services |
