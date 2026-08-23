# Documenso — Tester Environment

## Quick Start

```bash
./tester-env deploy    # Install deps, start services, migrate DB, seed data, launch dev server
./tester-env seed      # (no-op: deploy already seeds)
./tester-env verify    # Check app is responding at configured URL
./tester-env status    # Show Docker service status
./tester-env logs      # Tail app + service logs
./tester-env stop      # Stop services (preserves data)
./tester-env reset     # Stop services and remove volumes (full reset)
```

## Architecture

The CLI runs Documenso from source in development mode, using `npm run prisma:migrate-dev` + `TESTER_ENV=1 npm run prisma:seed` for deterministic setup. Services (PostgreSQL, Inbucket, Redis, MinIO) are managed via `docker compose` with the overlay at `docker/development/compose.tester-env.yml`.

### Service Ports

| Service  | Container Port | Host Port |
|----------|---------------|-----------|
| App      | 3000          | 3100      |
| Postgres | 5432          | 54320     |
| Inbucket | 9000          | 9005      |
| Redis    | 6379          | 63790     |
| MinIO    | 9002          | 9006      |

### Environment

Configured automatically via `setup_env()` in `tester-env`:
- `.env` copied from `.env.example` if missing
- `PORT`, `NEXT_PUBLIC_WEBAPP_URL`, `NEXT_PRIVATE_INTERNAL_WEBAPP_URL`, `NEXT_PRIVATE_UPLOAD_ENDPOINT` set automatically

### Deterministic Seed

Run with `TESTER_ENV=1` flag to produce deterministic data including admin user (`admin@documenso.com` / `password`).

### Browser Smoke

- Admin login at `http://host.docker.internal:3100/signin` (browser MCP containers reach the app via host.docker.internal; NEXT_PUBLIC_WEBAPP_URL is set to this origin so auth requests stay same-origin)
- Host-side health checks use `http://localhost:3100` (host.docker.internal does not resolve on the host)
- Seed populates templates, documents, and team data visible on dashboard
- Mail preview available at `http://host.docker.internal:9005` (Inbucket)

## Reset Path

```bash
./tester-env reset     # docker compose down -v — removes all containers and volumes
```

State is fully reproducible from a clean checkout.

## Baseline

Branch: `tester-env-baseline`
Upstream baseline: `8c0e029b1b2d954abd0b94a843561a49847d7a46` (feat: add pending signed PDF downloads)
Fork: https://github.com/Smartesting/documenso
