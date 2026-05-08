# AGENTS.md

## Repo overview

This repo contains **only Docker Compose files** for deploying two separate project management tools. There is no application source code, build system, or tests.

## Files

| File | Purpose |
|---|---|
| `docker-compose.yml` | **Kan** — lightweight kanban board (`ghcr.io/kanbn/kan`) with Postgres |
| `plane.docker-compose.yml` | **Plane** — full-featured PM tool deployed on Dokploy |
| `plan.docker-compose.yml` | Old/draft Plane compose using `build:` from source — do not use |

## Deployment target: Dokploy

Both active compose files are deployed via **Dokploy**, which runs **Traefik** as a built-in reverse proxy owning port 80/443.

**Critical constraints:**
- Never bind `ports: 80` or `ports: 443` — Traefik already owns them. Any service that tries will fail with `port is already allocated`.
- Never bind any port that may already be in use on the host (e.g. 3000). Prefer no host port binding and let Dokploy route via Docker network.
- Plane's built-in `proxy` service is intentionally disabled in `plane.docker-compose.yml` for this reason.

## Plane compose (`plane.docker-compose.yml`)

- Uses **pre-built images** from Docker Hub (`makeplane/plane-*:stable`). No source code needed in repo.
- Services: `web`, `space`, `admin`, `live`, `api`, `worker`, `beat-worker`, `migrator`, `plane-db`, `plane-redis`, `plane-mq`, `plane-minio`
- `migrator` runs once (`restart: on-failure`) before other services use the DB
- Domain routing: configure in Dokploy → Domains tab, point to `web` container port `3000`
- Required env vars to set in Dokploy (no defaults are safe for production):
  - `APP_DOMAIN` — your actual domain
  - `WEB_URL` and `CORS_ALLOWED_ORIGINS` — must match domain
  - `SECRET_KEY` — change from default
  - `LIVE_SERVER_SECRET_KEY` — set a secure random string
  - `POSTGRES_PASSWORD`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`

## Kan compose (`docker-compose.yml`)

- Required env vars: `POSTGRES_URL`, `POSTGRES_PASSWORD`, `BETTER_AUTH_SECRET`, `NEXT_PUBLIC_BASE_URL`
- `web` defaults to port `3001` (configurable via `WEB_PORT`)
- `migrate` must complete successfully before `web` starts — enforced via `depends_on: condition: service_completed_successfully`
- Port 5432 is exposed for Postgres — verify it's not already allocated on host
