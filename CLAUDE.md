# Repository Guide for Coding Agents

## Project context

TravBuddy is an open-source backend for AI trip planning, travel search and
recommendations, video-to-location processing, marketplace workflows, and
optional HostOS / StayOS hotel operations. The hosted services were retired;
do not assume production endpoints, credentials, or cloud resources are
available. See [SHUTDOWN.md](./SHUTDOWN.md).

The default runtime is the travel application. HostOS is optional and uses a
separate Supabase project configuration. The backend uses Supabase APIs
throughout; a plain PostgreSQL URL is not a drop-in replacement.

## Before changing code

1. Read the relevant implementation, tests, settings, and schema/migrations.
2. Trace the existing call path and reuse established helpers.
3. Keep changes scoped to the requested behavior; preserve existing API shapes
   unless the task explicitly changes them.
4. Add focused tests for new or changed behavior. Avoid tests that require live
   provider accounts unless they are explicitly integration tests.
5. For database changes, update the appropriate migration and related schema
   documentation. Consider row-level security and both deployment modes.

## Repository map

- `app/api/`: FastAPI endpoints, grouped by API version and domain.
- `app/agents/`, `app/trips_doc/`: trip planning and TripDoc workflows.
- `app/services/`: provider clients, business logic, PMS, and messaging.
- `app/reco/`: recommendation scoring and retrieval.
- `app/db/`, `app/storage/`: Supabase and cloud-storage access.
- `app/workers/`: Celery tasks and scheduled work.
- `schema.sql`, `migrations/`: database schema and incremental changes.
- `tests/`: unit and integration tests.
- `docs/`: architecture and engineering design documents.

## Local commands

PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pytest -q
$env:EXPOSE_API_DOCS = "true"
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

Swagger is disabled by default. Local API docs are available at `/docs` when
`EXPOSE_API_DOCS=true`. Supabase, Redis, Vertex AI, and other integrations need
their own local configuration; see [integration.md](./integration.md).

## Safety and configuration

- Never add real API keys, service-role credentials, JWT secrets, webhook
  secrets, or private-key material to source files or documentation.
- Keep secrets in ignored local `.env` files or a deployment secret manager.
- Do not send live payment, messaging, or provider requests from tests.
- Do not run destructive migrations or modify hosted data without explicit
  authorization.
- Keep `/docs`, `/redoc`, and `/openapi.json` disabled in production by default.
- Surface external-service failures clearly; do not mask them with success-like
  fallback responses.

## Documentation references

- [README.md](./README.md): project overview, setup, endpoints, and structure.
- [product.md](./product.md): product scope and status.
- [Features.md](./Features.md): capability catalogue.
- [technicals.md](./technicals.md): architecture and local development.
- [integration.md](./integration.md): external services and configuration.
- [Feature guide index](./docs/features/index.md): feature behavior by domain.
- [API surface](./docs/api_surface.md): route-family reference.
- [Database schema](./docs/database_schema.md): entities, relationships, and policies.
- [Database migrations](./docs/database_migrations.md): migration history and safety notes.
- [Workers and schedules](./docs/workers_and_schedules.md): Celery tasks and queue ownership.
- [Security and tenancy](./docs/security_and_tenancy.md): auth and authorization boundaries.
- [Codebase map](./docs/codebase_map.md): source layout, tests, and scripts.
- [Recommendation engine](./docs/recommendation_engine.md)
- [Trip engine architecture](./docs/trip_engine_architecture.md)
- [Persistence architecture](./docs/persistence_architecture.md)
