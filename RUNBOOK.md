# Operator and developer runbook

## Development prerequisites

- Node.js `22.22.2` or later in the 22.x line, 24.x, or 26+. Node.js 23.x and 25.x are unsupported.
- pnpm `9.15.0` (through Corepack), Docker Engine, and Docker Compose.

Install dependencies and generate the Prisma client:

```bash
corepack pnpm install --frozen-lockfile
corepack pnpm --filter @rakazo/db generate
```

Generation writes the ignored client under `packages/db/src/generated/prisma`; it generates code and does not apply database migrations.

## Local checks

These are available commands; this runbook does not claim they have been run:

```bash
corepack pnpm exec vitest run packages/adapters/src/executor-readonly-approval.test.ts
corepack pnpm --filter @rakazo/adapters check
corepack pnpm test
corepack pnpm lint
corepack pnpm check
corepack pnpm build
corepack pnpm format
```

`format` writes formatting changes; review its scope before using it.

## Production Compose topology

The production Compose stack keeps the API on internal port `3100` and web on internal port `5173`. Caddy publishes host ports `80` and `443`. The deployment hostname is deployment-specific; do not assume a URL from this runbook. The Compose API health check probes `/health`; web depends on a healthy API.

The one-shot `migrate` service runs `pnpm db:deploy` as the database owner before API startup. API and worker use the separate application login. Follow [self-host configuration](docs/self-host-configuration.md#database-roles) for database-role details.

## Candidate images and deployment

Image publication is governed by [publish-server-image.yml](.github/workflows/publish-server-image.yml). Applicable CI workflows publish source-addressed `sha-<full-commit>` images; see [the upgrade guide](docs/self-host-upgrading.md#published-images-and-tags) for tag behavior.

The production deploy launcher fetches `origin/main`; a requested deployment revision must be a full commit SHA and an ancestor of `origin/main`. The deploy script resets the deployment checkout to that requested revision (or `origin/main` when none is requested), runs migrations, then deploys the application. CI publication/deployment sequencing is the release path. An uncommitted local candidate is not published or deployable and must not be presented as a release candidate.

## Upgrades and rollback

Use [docs/self-host-upgrading.md](docs/self-host-upgrading.md) as the authoritative procedure for upgrades, Postgres 18 migration, and rollback. Application rollback does not reverse database migrations. Retain the previous release/image as required by that guide; do not prune the previous application image before the next update is accepted.

**Persistent data warning:** `docker compose down -v` deletes Postgres state. Never use it for a persistent environment. The deploy script performs bounded Docker image and builder-cache cleanup only after a healthy rollout and preserves/tag-retains previous images for rollback; do not run cleanup manually without following the documented procedure.

## NOT VERIFIED

- Actual production URL or deployment target.
- Local runtime availability, fixtures or test data.
- Any shutdown or cleanup execution, deployment result, or migration result.

No server, container, database, migration, fixture, shutdown, or cleanup operation occurred while preparing this runbook. For secret handling, follow the repository's self-host documentation; never place credential values in this runbook.
