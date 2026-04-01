# Tasks: Remove SHL Components

**Input**: Design documents from `/specs/002-remove-shl-components/`
**Prerequisites**: plan.md (required), spec.md (required), research.md, data-model.md, quickstart.md

**Tests**: No tests requested. Verification is manual via `docker compose config` and grep sweep (see quickstart.md).

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Repository root**: `base/`, `dev/`, `prod/` directories containing Docker Compose files
- **Keycloak config**: `base/config/keycloak/import/tot-realm.json`

---

## Phase 1: Setup

**Purpose**: No setup required — this feature removes existing configuration rather than adding new infrastructure.

*(No tasks in this phase)*

---

## Phase 2: Foundational

**Purpose**: No foundational work required — all changes are independent removals within existing files.

*(No tasks in this phase)*

---

## Phase 3: User Story 1 — Remove SHL services from deployment (Priority: P1) 🎯 MVP

**Goal**: Remove shl-creator and shl-server service definitions from all Docker Compose configuration files (base, dev, prod) so no SHL containers are created on deployment.

**Independent Test**: Run `docker compose config --quiet` in dev/ and prod/ directories to confirm valid YAML. Verify no `shl-creator` or `shl-server` services appear in the rendered config.

### Implementation for User Story 1

- [x] T001 [US1] Remove the `shl-creator` service block from `base/docker-compose.yaml`
- [x] T002 [US1] Remove the `shl-server` service block from `base/docker-compose.yaml`
- [x] T003 [P] [US1] Remove the `shl-creator` service block from `dev/docker-compose.yaml`
- [x] T004 [P] [US1] Remove the `shl-server` service block from `dev/docker-compose.yaml`
- [x] T005 [P] [US1] Remove the `shl-creator` service block from `prod/docker-compose.yaml`
- [x] T006 [P] [US1] Remove the `shl-server` service block from `prod/docker-compose.yaml`

**Checkpoint**: At this point, deploying the environment should create zero SHL containers. All remaining services should still be defined correctly.

---

## Phase 4: User Story 2 — Remove SHL references from remaining services (Priority: P1)

**Goal**: Remove all environment variables and configuration entries in cpro, keycloak, and fhir-auth that reference SHL services, and remove the `shl_creator` OIDC client from the Keycloak realm import.

**Independent Test**: Verify cpro has no `SHL_MANAGER_URL`, keycloak has no `KEYCLOAK_SHL_CREATOR_*` variables, fhir-auth has no SHL-related TODO comments, and `tot-realm.json` has no `shl_creator` client.

### Implementation for User Story 2

- [x] T007 [US2] Remove `SHL_MANAGER_URL` environment variable from the cpro service in `base/docker-compose.yaml`
- [x] T008 [US2] Remove `KEYCLOAK_SHL_CREATOR_BASE` and `KEYCLOAK_SHL_CREATOR_POST_LOGOUT_REDIRECT_URL` environment variables from the keycloak service in `base/docker-compose.yaml`
- [x] T009 [US2] Remove the TODO comment referencing shl-creator from the fhir-auth service labels in `base/docker-compose.yaml`
- [x] T010 [US2] Remove the `shl_creator` OIDC client object from the `clients` array in `base/config/keycloak/import/tot-realm.json`

**Checkpoint**: At this point, no remaining service references SHL components. Keycloak realm import no longer includes the shl_creator client.

---

## Phase 5: User Story 3 — Clean up SHL-related artifacts (Priority: P2)

**Goal**: Remove all residual SHL files, volume definitions, and image tag variable references so the project contains no remnants of SHL components.

**Independent Test**: Run `grep -ri 'shl' base/ dev/ prod/ --include='*.yaml' --include='*.json' --include='*.default'` and confirm zero results.

### Implementation for User Story 3

- [x] T011 [P] [US3] Delete the file `dev/shl-creator.env.default`
- [x] T012 [P] [US3] Delete the file `prod/shl-creator.env.default`
- [x] T013 [P] [US3] Remove the `shl-server-data` volume definition from `dev/docker-compose.yaml`
- [x] T014 [P] [US3] Remove the `shl-server-data` volume definition from `prod/docker-compose.yaml`
- [x] T015 [US3] Remove any `SHL_CREATOR_IMAGE_TAG` or `SHL_SERVER_IMAGE_TAG` variable references from `base/docker-compose.yaml` (if present)

**Checkpoint**: The entire project should now have zero references to SHL components in any active configuration file.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final verification across all changes

- [x] T016 Run `docker compose config --quiet` validation in `dev/` directory to confirm valid compose configuration
- [x] T017 Run `docker compose config --quiet` validation in `prod/` directory to confirm valid compose configuration
- [x] T018 Run full grep sweep per quickstart.md: `grep -ri 'shl' base/ dev/ prod/ --include='*.yaml' --include='*.json' --include='*.default'` to confirm zero SHL references remain

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: Empty — no setup required
- **Foundational (Phase 2)**: Empty — no foundational work required
- **User Story 1 (Phase 3)**: Can start immediately — removes service definitions
- **User Story 2 (Phase 4)**: Can start immediately — removes references in remaining services. Independent of US1 (different sections of same files)
- **User Story 3 (Phase 5)**: Can start immediately for file deletions (T011, T012). Volume removals (T013, T014) should follow US1 completion to avoid editing same file sections simultaneously
- **Polish (Phase 6)**: Depends on ALL user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: No dependencies — can start immediately
- **User Story 2 (P1)**: No dependencies on US1 — touches different sections of `base/docker-compose.yaml`
- **User Story 3 (P2)**: File deletions (T011, T012) have no dependencies. Volume/variable tasks (T013-T015) are safest after US1 completes

### Within Each User Story

- T001 and T002 share `base/docker-compose.yaml` — execute sequentially
- T003-T006 touch different files — can run in parallel
- T007-T009 share `base/docker-compose.yaml` — execute sequentially
- T010 touches a different file (`tot-realm.json`) — can run in parallel with T007-T009
- T011-T014 touch different files — can all run in parallel

### Parallel Opportunities

- US1 tasks T003, T004, T005, T006 can run in parallel (different files)
- US2 task T010 can run in parallel with T007-T009 (different file)
- US3 tasks T011, T012, T013, T014 can all run in parallel (different files)
- US1 and US2 can proceed in parallel (different sections of shared files)

---

## Parallel Example: User Story 1

```bash
# Sequential (same file - base/docker-compose.yaml):
Task T001: "Remove shl-creator service block from base/docker-compose.yaml"
Task T002: "Remove shl-server service block from base/docker-compose.yaml"

# Then parallel (different files):
Task T003: "Remove shl-creator service block from dev/docker-compose.yaml"
Task T004: "Remove shl-server service block from dev/docker-compose.yaml"
Task T005: "Remove shl-creator service block from prod/docker-compose.yaml"
Task T006: "Remove shl-server service block from prod/docker-compose.yaml"
```

## Parallel Example: User Story 3

```bash
# All parallel (different files):
Task T011: "Delete dev/shl-creator.env.default"
Task T012: "Delete prod/shl-creator.env.default"
Task T013: "Remove shl-server-data volume from dev/docker-compose.yaml"
Task T014: "Remove shl-server-data volume from prod/docker-compose.yaml"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 3: User Story 1 — remove all SHL service definitions
2. **STOP and VALIDATE**: Run `docker compose config --quiet` in dev/ and prod/
3. Deploy/demo if ready — environment runs without SHL containers

### Incremental Delivery

1. User Story 1 → Remove service definitions → Validate compose → Deploy (MVP!)
2. User Story 2 → Remove stale references → Validate no warnings → Deploy
3. User Story 3 → Delete residual files/volumes → Full grep sweep → Deploy (Complete!)
4. Each story adds cleanup value without breaking previous removals

### Single Developer Strategy (Recommended)

Since all changes are in a small number of files, the most efficient approach is:

1. Complete US1 + US2 together (both modify `base/docker-compose.yaml`)
2. Complete US3 (file deletions and volume cleanup)
3. Run full verification (Phase 6)

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- No tests phase — verification is via compose config validation and grep sweep
- This is a removal-only feature: no new code, models, or services are introduced
- Commit after each user story phase for clean git history
- Stop at any checkpoint to validate independently
