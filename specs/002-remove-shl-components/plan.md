# Implementation Plan: Remove SHL Components

**Branch**: `002-remove-shl-components` | **Date**: 2026-03-31 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/002-remove-shl-components/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Remove the `shl-creator` and `shl-server` services from all Docker Compose configuration files (base, dev, prod), remove all references to these services from remaining service configurations (cpro, keycloak, fhir-auth), delete related env defaults and volume definitions, and remove the `shl_creator` OIDC client from the Keycloak realm import.

## Technical Context

**Language/Version**: Docker Compose v3 (YAML configuration), JSON (Keycloak realm import)
**Primary Dependencies**: Docker Compose `extends` pattern (base/dev/prod layering)
**Storage**: N/A (infrastructure-only change)
**Testing**: Manual verification via `docker compose config` validation and grep-based sweep
**Target Platform**: Linux server (Docker host)
**Project Type**: Docker Compose infrastructure repository
**Performance Goals**: N/A (no runtime change to remaining services)
**Constraints**: Must not break any remaining service (cpro, mysql, db, fhir, keycloak, fhir-auth)
**Scale/Scope**: 8 files to modify, 2 files to delete

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Microservices via Docker Compose | PASS | Removing unused services; remaining services still follow pattern |
| II. Environment Separation | PASS | Changes applied consistently across base/, dev/, prod/ layers |
| III. Network Isolation | PASS | No network topology changes; shl services were on ingress only |
| IV. Security-First | PASS | Removing unused OIDC client reduces attack surface |
| V. Pull-Based Deployment | PASS | No build changes |
| VI. Configuration as Code | PASS | Keycloak realm import updated in version control |
| VII. Sensitive Data Management | PASS | Removing shl-creator.env.default files; no new secrets introduced |
| VIII. Health Checks | PASS | No health check changes |
| IX. Git Submodules | PASS | No submodule changes |
| X. Dev-Prod Parity | PASS | Same removals applied to both dev and prod |

**Gate result**: PASS - All principles satisfied. Removal of unused services is fully aligned with the constitution.

## Project Structure

### Documentation (this feature)

```text
specs/002-remove-shl-components/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
base/
├── docker-compose.yaml          # Remove shl-creator, shl-server services + SHL env vars
└── config/keycloak/import/
    └── tot-realm.json           # Remove shl_creator OIDC client

dev/
├── docker-compose.yaml          # Remove shl-creator, shl-server service extensions + shl-server-data volume
├── shl-creator.env.default      # DELETE this file
└── (keycloak.env.default)       # No SHL refs - no change needed
└── (cpro.env.default)           # No SHL refs - no change needed

prod/
├── docker-compose.yaml          # Remove shl-creator, shl-server service extensions + shl-server-data volume
└── shl-creator.env.default      # DELETE this file
```

## Complexity Tracking

> No violations. All changes align with constitution principles.
