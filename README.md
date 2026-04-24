# images

Custom Docker images built and published to [ghcr.io/yoyozbi](https://ghcr.io/yoyozbi) via GitHub Actions.

## Images

### `postgres-backup`

**Image:** `ghcr.io/yoyozbi/images/postgres-backup`

Based on `postgres:17-alpine`, this image bundles everything needed to back up a PostgreSQL 17 database: the matching `pg_dump`/`pg_restore` binaries (same major version as the target server), `curl` for shipping dumps to remote storage, and `jq` for processing JSON responses from backup APIs.

Using the same major version as the target server avoids `pg_dump` compatibility warnings and ensures the dump format is fully supported.

### `super-sync`

**Image:** `ghcr.io/yoyozbi/images/super-sync`

A self-hostable sync server for [Super Productivity](https://github.com/super-productivity/super-productivity), built from the upstream `packages/super-sync-server` source. It exposes a REST API that the Super Productivity client uses to synchronise tasks across devices via an append-only operation log backed by PostgreSQL.

**Patch applied:** The upstream project was bootstrapped with `prisma db push` instead of `prisma migrate dev`, so no initial `CREATE TABLE` migration was ever committed. The upstream Dockerfile runs `prisma migrate deploy` on startup, which would fail on a fresh database because all existing migrations are `ALTER TABLE` statements that assume the base tables already exist.

The build workflow injects a baseline migration (`20200101000000_init`) before the Docker build that creates the four core tables (`users`, `operations`, `user_sync_state`, `sync_devices`) in the state they were in before the first upstream incremental migration. This makes the image fully self-initialising: just point it at an empty PostgreSQL database and it will bootstrap the schema automatically on first start.

## Usage

See each image's workflow in [`.github/workflows/`](.github/workflows/) for build inputs and tags. Images are triggered manually via `workflow_dispatch` with a `ref` input (tag or commit SHA from the upstream repo).
