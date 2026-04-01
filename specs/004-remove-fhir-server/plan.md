# Implementation Plan: Remove FHIR Server

**Branch**: `004-remove-fhir-server` | **Date**: 2026-03-31 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/004-remove-fhir-server/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Remove the HAPI FHIR server and its JWT authentication proxy from the TOT deployment. This involves deleting service definitions from all Docker Compose files (base, dev, prod), removing FHIR-related environment variables from cPRO, cleaning up the database initialization script, removing SMART-on-FHIR scopes from the Keycloak realm import, and deleting orphaned environment template files.

## Technical Context

**Language/Version**: Docker Compose v3 (YAML), JSON (Keycloak realm), SQL (db init)
**Primary Dependencies**: Docker Compose `extends` pattern (base/dev/prod layering)
**Storage**: PostgreSQL (shared with Keycloak — retained), MariaDB (cPRO — unaffected)
**Testing**: `docker compose config` validation, manual deployment verification
**Target Platform**: Linux server (Docker host)
**Project Type**: Infrastructure (Docker Compose deployment)
**Performance Goals**: N/A (removal reduces resource usage)
**Constraints**: Must not break remaining services (cPRO, Keycloak, MySQL, PostgreSQL, Traefik)
**Scale/Scope**: 6 files modified, 2 files deleted

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Microservices via Docker Compose | PASS | Removing services, not adding. Compose `extends` pattern preserved. |
| II. Environment Separation | PASS | Changes applied consistently across base/dev/prod layers. |
| III. Network Isolation | PASS | `fhir-internal` alias removed with service. `internal` and `ingress` networks retained. |
| IV. Security-First | PASS | Removing an attack surface (fewer exposed services). OIDC remains for cPRO/Keycloak. |
| V. Pull-Based Deployment | PASS | No local builds introduced. |
| VI. Configuration as Code | PASS | All changes are to version-controlled config files. |
| VII. Sensitive Data Management | PASS | Removing `fhir-auth.env.default` templates. No new secrets introduced. |
| VIII. Health Checks | PASS | FHIR health check removed with service. Remaining health checks (db) unaffected. |
| IX. Git Submodules | N/A | No submodule changes. |
| X. Dev-Prod Parity | PASS | Changes applied symmetrically to dev and prod. |

**Gate result**: PASS — no violations. No complexity tracking needed.

### Post-Design Re-check

All constitution principles remain satisfied after Phase 1 design. No new external interfaces introduced, no contracts directory needed.

## Project Structure

### Documentation (this feature)

```text
specs/004-remove-fhir-server/
├── plan.md              # This file
├── research.md          # Phase 0 output — 6 research decisions
├── data-model.md        # Phase 1 output — configuration object inventory
├── quickstart.md        # Phase 1 output — implementation guide
└── tasks.md             # Phase 2 output (created by /speckit.tasks)
```

### Source Code (repository root)

```text
base/
├── docker-compose.yaml          # Remove fhir + fhir-auth services, cPRO FHIR env vars
├── config/
│   ├── db/db.init.sql           # Remove hapifhir database creation
│   └── keycloak/import/
│       └── tot-realm.json       # Remove SMART-on-FHIR scopes
dev/
├── docker-compose.yaml          # Remove fhir + fhir-auth service extensions
├── fhir-auth.env.default        # DELETE
prod/
├── docker-compose.yaml          # Remove fhir + fhir-auth service extensions
├── fhir-auth.env.default        # DELETE
```

**Structure Decision**: Existing Docker Compose layered structure (base/dev/prod) preserved. No new directories or structural changes.

## Complexity Tracking

> No constitution violations — table not needed.
