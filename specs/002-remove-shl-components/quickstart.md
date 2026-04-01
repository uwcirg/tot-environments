# Quickstart: Remove SHL Components

**Feature**: 002-remove-shl-components | **Date**: 2026-03-31

## What This Change Does

Removes two unused Docker Compose services (`shl-creator`, `shl-server`) and all their configuration references from the TOT deployment.

## Files to Modify

1. **`base/docker-compose.yaml`** — Remove `shl-creator` and `shl-server` service blocks, remove `SHL_MANAGER_URL` from cpro, remove `KEYCLOAK_SHL_CREATOR_*` vars from keycloak, remove SHL-related TODO comment from fhir-auth
2. **`dev/docker-compose.yaml`** — Remove `shl-creator` and `shl-server` service blocks, remove `shl-server-data` volume
3. **`prod/docker-compose.yaml`** — Remove `shl-creator` and `shl-server` service blocks, remove `shl-server-data` volume
4. **`base/config/keycloak/import/tot-realm.json`** — Remove `shl_creator` client from `clients` array

## Files to Delete

5. **`dev/shl-creator.env.default`**
6. **`prod/shl-creator.env.default`**

## Verification

After making changes, run:

```bash
# Validate compose files parse correctly
cd dev && docker compose config --quiet && cd ..
cd prod && docker compose config --quiet && cd ..

# Confirm zero SHL references remain
grep -ri 'shl' base/ dev/ prod/ --include='*.yaml' --include='*.json' --include='*.default'
```

Expected: compose config succeeds silently, grep returns no results.
