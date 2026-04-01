# Tasks: Remove FHIR Server

**Input**: Design documents from `/specs/004-remove-fhir-server/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, quickstart.md

**Tests**: Not requested — no test tasks included.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup

**Purpose**: Establish baseline state before making changes

- [x] T001 Verify current Docker Compose configuration is valid by running `docker compose -f dev/docker-compose.yaml config` and `docker compose -f prod/docker-compose.yaml config`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: No foundational tasks needed — this feature removes existing configuration rather than building new infrastructure.

**Checkpoint**: Proceed directly to user story implementation.

---

## Phase 3: User Story 1 — Remove FHIR Server from Deployment (Priority: P1) 🎯 MVP

**Goal**: The deployment no longer starts a HAPI FHIR server or its JWT authentication proxy.

**Independent Test**: Run `docker compose -f dev/docker-compose.yaml config` and confirm no `fhir` or `fhir-auth` services appear in the output.

### Implementation for User Story 1

- [x] T002 [US1] Remove `fhir` service block from base/docker-compose.yaml (lines 81–113)
- [x] T003 [US1] Remove `fhir-auth` service block from base/docker-compose.yaml (lines 159–184)
- [x] T004 [P] [US1] Remove `fhir` service block from dev/docker-compose.yaml (lines 21–34)
- [x] T005 [P] [US1] Remove `fhir-auth` service block from dev/docker-compose.yaml (lines 45–50)
- [x] T006 [P] [US1] Remove `fhir` service block from prod/docker-compose.yaml (lines 21–26)
- [x] T007 [P] [US1] Remove `fhir-auth` service block from prod/docker-compose.yaml (lines 37–42)

**Checkpoint**: No FHIR or fhir-auth service definitions exist in any Docker Compose file. Remaining services (cPRO, Keycloak, db, mysql) are unaffected.

---

## Phase 4: User Story 2 — cPRO Application Functions Without FHIR (Priority: P1)

**Goal**: The cPRO service no longer references FHIR environment variables, so it starts without attempting FHIR connections.

**Independent Test**: Run `docker compose -f dev/docker-compose.yaml config` and confirm the `cpro` service has no `FHIR_R4_SERVER_ENDPOINT` or `FHIR_R4_EXTERNAL_ID_SYSTEM` environment variables.

### Implementation for User Story 2

- [x] T008 [US2] Remove `FHIR_R4_SERVER_ENDPOINT` and `FHIR_R4_EXTERNAL_ID_SYSTEM` environment variables (and associated comments) from the cPRO service in base/docker-compose.yaml (lines 17–20)

**Checkpoint**: cPRO service definition contains no FHIR-related environment variables.

---

## Phase 5: User Story 3 — Clean Up FHIR Database and Configuration Artifacts (Priority: P2)

**Goal**: All FHIR-specific configuration artifacts are removed — database initialization, Keycloak scopes, and environment template files.

**Independent Test**: Grep the entire repository for `fhir` (case-insensitive) and confirm zero matches outside `specs/`.

### Implementation for User Story 3

- [x] T009 [P] [US3] Remove `---- HAPI ----` comment and `create database hapifhir;` line from base/config/db/db.init.sql (lines 5–6)
- [x] T010 [P] [US3] Remove `patient/*.read` scope definition (lines 398–407) and `launch` scope definition (lines 409–419) from base/config/keycloak/import/tot-realm.json
- [x] T011 [US3] Remove `"launch"` and `"patient/*.read"` entries from `defaultOptionalClientScopes` array in base/config/keycloak/import/tot-realm.json (lines 794–795)
- [x] T012 [P] [US3] Delete dev/fhir-auth.env.default
- [x] T013 [P] [US3] Delete prod/fhir-auth.env.default

**Checkpoint**: No FHIR-related databases, scopes, or environment files remain. Fresh deployments create only the `keycloak` database.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final validation across all changes

- [x] T014 Validate dev compose config by running `docker compose -f dev/docker-compose.yaml config`
- [x] T015 Validate prod compose config by running `docker compose -f prod/docker-compose.yaml config`
- [x] T016 Grep entire repository for `fhir` (case-insensitive) and confirm zero matches outside `specs/` directory
- [x] T017 Verify remaining services in compose output: cpro, mysql, db, keycloak (+ traefik from external)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Foundational (Phase 2)**: Skipped — no foundational work needed
- **User Story 1 (Phase 3)**: Depends on Setup; removes service definitions
- **User Story 2 (Phase 4)**: Depends on Setup; independent of US1 (different lines in base compose)
- **User Story 3 (Phase 5)**: Depends on Setup; independent of US1/US2 (different files)
- **Polish (Phase 6)**: Depends on all user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Phase 1 — no dependencies on other stories
- **User Story 2 (P1)**: Can start after Phase 1 — independent of US1 (edits different lines in same file)
- **User Story 3 (P2)**: Can start after Phase 1 — fully independent (different files entirely)

### Within Each User Story

- T002 must complete before T003 (same file, adjacent blocks in base compose)
- T004–T007 can run in parallel (different files: dev vs prod compose)
- T009–T013 can mostly run in parallel (different files), except T010 before T011 (same file, same JSON structure)

### Parallel Opportunities

- US1, US2, and US3 can all proceed in parallel (different files or non-overlapping regions)
- Within US1: T004+T005 (dev) can run in parallel with T006+T007 (prod)
- Within US3: T009, T012, T013 can run in parallel (different files)
- All Polish tasks (T014–T017) can run in parallel after all stories complete

---

## Parallel Example: User Story 1

```bash
# After T002 and T003 complete (base compose, sequential — same file):
# Launch dev and prod removals in parallel:
Task: "Remove fhir service block from dev/docker-compose.yaml"       # T004
Task: "Remove fhir-auth service block from dev/docker-compose.yaml"  # T005
Task: "Remove fhir service block from prod/docker-compose.yaml"      # T006
Task: "Remove fhir-auth service block from prod/docker-compose.yaml" # T007
```

## Parallel Example: User Story 3

```bash
# All different files — launch in parallel:
Task: "Remove hapifhir database creation from db.init.sql"           # T009
Task: "Delete dev/fhir-auth.env.default"                             # T012
Task: "Delete prod/fhir-auth.env.default"                            # T013
# Then T010 (scope definitions) before T011 (scope references) — same file
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (baseline validation)
2. Complete Phase 3: User Story 1 (remove FHIR service definitions)
3. **STOP and VALIDATE**: Run `docker compose config` — no FHIR services should appear
4. Deploy/demo if ready — system runs without FHIR containers

### Incremental Delivery

1. Complete US1 → FHIR containers gone → Validate (MVP!)
2. Add US2 → cPRO clean of FHIR references → Validate
3. Add US3 → All artifacts removed → Validate
4. Polish → Full verification pass
5. Each story adds cleanup value without breaking previous stories

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Line numbers reference current file state; verify before editing if other tasks have already modified the file
- No tests were requested in the specification
- Commit after each phase or logical group
- Stop at any checkpoint to validate independently
