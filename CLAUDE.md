# tot-environments Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-03-31

## Active Technologies
- Docker Compose v3 (YAML), JSON (Keycloak realm), SQL (db init) + Docker Compose `extends` pattern (base/dev/prod layering) (004-remove-fhir-server)
- PostgreSQL (shared with Keycloak — retained), MariaDB (cPRO — unaffected) (004-remove-fhir-server)

- Docker Compose v3 (YAML configuration), JSON (Keycloak realm import) + Docker Compose `extends` pattern (base/dev/prod layering) (002-remove-shl-components)

## Project Structure

```text
src/
tests/
```

## Commands

# Add commands for Docker Compose v3 (YAML configuration), JSON (Keycloak realm import)

## Code Style

Docker Compose v3 (YAML configuration), JSON (Keycloak realm import): Follow standard conventions

## Recent Changes
- 004-remove-fhir-server: Added Docker Compose v3 (YAML), JSON (Keycloak realm), SQL (db init) + Docker Compose `extends` pattern (base/dev/prod layering)

- 002-remove-shl-components: Added Docker Compose v3 (YAML configuration), JSON (Keycloak realm import) + Docker Compose `extends` pattern (base/dev/prod layering)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
