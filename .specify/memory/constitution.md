<!--
  Sync Impact Report
  ===================
  Version change: N/A (initial) -> 1.0.0
  Modified principles: N/A (initial population)
  Added sections:
    - Core Principles (10 architecture principles)
    - Core Services (service inventory)
    - Best Practices (5 operational practices)
    - Governance
  Removed sections: None
  Templates requiring updates:
    - .specify/templates/plan-template.md — ✅ no updates needed (Constitution Check
      section is dynamically filled from this file)
    - .specify/templates/spec-template.md — ✅ no updates needed (generic structure)
    - .specify/templates/tasks-template.md — ✅ no updates needed (generic structure)
  Follow-up TODOs: None
-->

# Take on Transplant (TOT) Constitution

Take on Transplant ("TOT") is a Docker Compose-based infrastructure repository
that deploys a public-facing, secure consumer health platform intended to
educate individuals with cystic fibrosis in their decisions concerning lung
transplant procedures. It collects patient-reported outcomes and provides
tailored education to users. Maintained by the University of Washington
Clinical Informatics Research Group (https://github.com/uwcirg).

## Core Principles

### I. Microservices via Docker Compose

Each service MUST run as a separate container, coordinated through Docker
Compose files. Compose `extends` MUST be used for DRY configuration — shared
service definitions live in `base/` and are extended by environment-specific
files.

### II. Environment Separation

`base/` holds shared service definitions. `dev/`, `prod/`, and `logs/` extend
base with environment-specific overrides. New environments MUST follow this
layering pattern rather than duplicating base definitions.

### III. Network Isolation

Two networks MUST be maintained:
- **ingress**: Internet-facing traffic routed through Traefik.
- **internal**: Databases and backing services with no external access.

Services MUST NOT join the ingress network unless they require external access.

### IV. Security-First

- OIDC authentication MUST be enforced for all user-facing services.
- TLS MUST terminate at the Traefik edge proxy.
- Credentials MUST be kept out of version control via `.env` files and
  `.gitignore`.

### V. Pull-Based Deployment

No local image builds. All container images MUST be pulled from GitHub
Container Registry (ghcr.io). Dockerfiles for local builds MUST NOT be
introduced into this repository.

### VI. Configuration as Code

Keycloak realm/user imports, database initialization scripts, and service
configurations MUST be version-controlled in `base/config/`. Runtime
configuration that cannot be committed (secrets) is handled per Principle VII.

### VII. Sensitive Data Management

Template files (`*.default`) MUST be committed for every `.env` file. Actual
`.env` files containing secrets MUST be gitignored and created per-deployment.
The `.gitignore` MUST explicitly list all secret-bearing file patterns.

### VIII. Health Checks

Services with startup dependencies (PostgreSQL, FHIR servers) MUST define
explicit health check configurations. `depends_on` conditions MUST reference
these health checks where supported by the Compose version.

### IX. Git Submodules for External Dependencies

External dependencies (e.g., logserver) MUST be managed as Git submodules
rather than vendored or copied into the repository. Submodule references MUST
point to stable commits or tags.

### X. Dev-Prod Parity

The development environment MUST support live code mounting via
`CPRO_CHECKOUT_DIR` with host user ID synchronization. Development compose
overrides MUST keep the dev environment as close to production as possible,
differing only in volume mounts and debug settings.

## Core Services

| Service | Description | Backend |
|---------|-------------|---------|
| **cPRO** | PHP-based clinical application for collecting patient-reported outcomes and delivering tailored content | MariaDB |
| **Keycloak (v22)** | Identity provider — OIDC/OAuth2, role-based access (staff + patient roles) | PostgreSQL |
| **Traefik (v2.2)** | Reverse proxy with Let's Encrypt TLS | N/A |
| **Logserver** | Centralized logging with PostgREST API and OAuth2-protected access | PostgreSQL |

## Best Practices

- **No secrets in version control** — Strict `.gitignore` for `.env` files;
  templates provided as `*.default`.
- **Least privilege networking** — Only services that need external access
  join the ingress network.
- **Password policy enforcement** — Keycloak MUST be configured with minimum
  8 characters, mixed case, digits, and password history.
- **Explicit service dependencies** — `depends_on` with health check
  conditions where supported.
- **Composable configuration** — Multiple compose files can be layered via
  the `COMPOSE_FILE` environment variable for flexible deployments.

## Governance

1. This constitution supersedes ad-hoc practices. All changes to the
   repository MUST be consistent with these principles.
2. Amendments to this constitution require:
   - A documented rationale for the change.
   - Review and approval via pull request.
   - A migration plan if the amendment invalidates existing configuration.
3. Versioning follows semantic versioning:
   - **MAJOR**: Principle removals or backward-incompatible redefinitions.
   - **MINOR**: New principles added or existing guidance materially expanded.
   - **PATCH**: Clarifications, wording fixes, non-semantic refinements.
4. Compliance review: PRs that modify `base/`, network definitions, or
   security-related configuration MUST reference the relevant principle(s)
   in review.

**Version**: 1.0.0 | **Ratified**: 2026-03-31 | **Last Amended**: 2026-03-31
