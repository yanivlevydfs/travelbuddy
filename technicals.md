# TravBuddy Technical Overview

## Architecture

TravBuddy is an async Python backend. FastAPI serves HTTP APIs; Celery workers
run long-lived or scheduled jobs; Supabase provides PostgreSQL, authentication,
and storage-facing APIs.

```text
Clients
  |
  v
FastAPI (app/main.py)
  |-- Supabase: PostgreSQL, Auth, pgvector
  |-- Vertex AI / Gemini and Google Places
  |-- Travel search providers and payment/email services
  |
  +-- Redis broker/cache/locks --> Celery workers and beat
  +-- Google Cloud Storage for optional media assets
```

## Main components

| Component | Location | Responsibility |
| --- | --- | --- |
| Application and settings | `app/main.py`, `app/config.py` | FastAPI setup, middleware, lifecycle, configuration |
| API routes | `app/api/v1/`, `app/api/v3/` | Versioned HTTP interfaces |
| Agent and trip planning | `app/agents/`, `app/trips_doc/`, `app/pipeline/` | Planning, orchestration, and trip generation |
| Search and recommendations | `app/services/`, `app/reco/` | Provider adapters, scoring, reranking, feedback |
| Persistence | `app/db/`, `app/storage/`, `schema.sql`, `migrations/` | Supabase clients, data access, schema changes |
| Background processing | `app/workers/` | Celery tasks, job processing, and scheduled work |
| Hotel operations | `app/services/pms/`, `app/services/messaging/` | HostOS workflows and guest communications |

## Runtime patterns

- Async HTTP handlers use background jobs where work may take longer than a
  request, including Reel processing and selected recommendation workflows.
- Redis is used by Celery and other coordination or caching features.
- Supabase access is wrapped in `app/db/client.py`; the application uses the
  Supabase client API rather than a generic PostgreSQL ORM.
- HostOS is an optional runtime mode and is designed to use a distinct
  Supabase project from the travel application.
- External integrations are configured through environment-backed settings in
  `app/config.py`.

## Stack

- Python 3.11+, FastAPI, Uvicorn, Pydantic v2
- Supabase PostgreSQL, Auth, and `pgvector`
- Celery and Redis
- Google GenAI SDK with Vertex AI, Google Maps/Places, and optional Cloud Storage
- `yt-dlp`, PySceneDetect, OpenCV, Pillow, and ImageHash for video processing
- `httpx`, `curl-cffi`, `structlog`, and Sentry SDK

See [requirements.txt](./requirements.txt) for pinned Python dependencies.

## Local development

From the repository root in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env
```

Review `.env` and set only credentials for integrations you intend to use.
`EXPOSE_API_DOCS=true` enables local Swagger UI at `/docs`; docs are disabled
by default. Start the API with:

```powershell
$env:EXPOSE_API_DOCS = "true"
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

Redis and Supabase are needed for their respective features. Vertex AI
credentials are needed for Gemini-backed workflows. The API may start without
these services, but its health endpoint will report missing dependencies and
dependent routes will not be fully functional.

## Data and schema

The base schema is in [schema.sql](./schema.sql); incremental changes are in
[migrations/](./migrations/). Apply database changes to the intended Supabase
project and review row-level security policies before deploying. StayOS and the
travel application use separate project configurations.

Related design references:

- [Database schema](./docs/database_schema.md) and [migration index](./docs/database_migrations.md)
- [API surface](./docs/api_surface.md)
- [Workers and schedules](./docs/workers_and_schedules.md)
- [Codebase map](./docs/codebase_map.md)
- [Recommendation engine](./docs/recommendation_engine.md)
- [Trip engine architecture](./docs/trip_engine_architecture.md)
- [Persistence architecture](./docs/persistence_architecture.md)
- [Agent architecture map](./docs/agent_architecture_map.md)

## Tests

Run the test suite with `python -m pytest -q` after installing pytest in the
active environment. Some tests or scripts require configured Supabase, Redis,
or other external services; use the relevant test documentation and fixtures
when running integration tests.
