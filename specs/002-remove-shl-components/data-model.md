# Data Model: Remove SHL Components

**Feature**: 002-remove-shl-components | **Date**: 2026-03-31

## Overview

This feature involves no new data entities. It removes configuration artifacts related to two Docker Compose services. The "data model" here describes the configuration objects being removed.

## Entities Removed

### Service: shl-creator (Docker Compose)

- **Location**: `base/docker-compose.yaml` (lines 190-208), `dev/docker-compose.yaml` (lines 52-57), `prod/docker-compose.yaml` (lines 44-49)
- **Image**: `ghcr.io/uwcirg/shl-tot`
- **Networks**: ingress only
- **Dependencies**: None within compose; depends on shl-server and keycloak externally
- **Env file**: `shl-creator.env` (from `shl-creator.env.default` template)

### Service: shl-server (Docker Compose)

- **Location**: `base/docker-compose.yaml` (lines 210-221), `dev/docker-compose.yaml` (lines 59-62), `prod/docker-compose.yaml` (lines 51-54)
- **Image**: `ghcr.io/uwcirg/shl-tot-server`
- **Networks**: ingress only
- **Volumes**: `shl-server-data` mounted at `/app/db`

### OIDC Client: shl_creator (Keycloak)

- **Location**: `base/config/keycloak/import/tot-realm.json` (within `clients` array)
- **Client ID**: `shl_creator`
- **Type**: Public OIDC client with standard flow
- **Scopes**: web-origins, acr, roles, profile, email (default); SMART-on-FHIR scopes (optional)

### Environment Variables Removed

| Variable | Service | File |
|----------|---------|------|
| `SHL_MANAGER_URL` | cpro | `base/docker-compose.yaml` |
| `KEYCLOAK_SHL_CREATOR_BASE` | keycloak | `base/docker-compose.yaml` |
| `KEYCLOAK_SHL_CREATOR_POST_LOGOUT_REDIRECT_URL` | keycloak | `base/docker-compose.yaml` |

### Files Deleted

| File | Purpose |
|------|---------|
| `dev/shl-creator.env.default` | Env template for shl-creator in dev |
| `prod/shl-creator.env.default` | Env template for shl-creator in prod |

### Volumes Removed

| Volume | Service | Files |
|--------|---------|-------|
| `shl-server-data` | shl-server | `dev/docker-compose.yaml`, `prod/docker-compose.yaml` |

## Relationships

```text
shl-creator ──uses──> shl-server (API)
shl-creator ──authenticates-via──> keycloak (shl_creator OIDC client)
cpro ──links-to──> shl-creator (SHL_MANAGER_URL)
keycloak ──configures──> shl_creator client (KEYCLOAK_SHL_CREATOR_BASE vars)
```

All four relationships are severed by this removal. No remaining service has a dependency on any SHL component.
