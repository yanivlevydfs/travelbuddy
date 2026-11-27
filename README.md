# TravBuddy Backend

An async **FastAPI** backend powering AI-driven multi-city trip planning, live travel search and recommendations, Instagram Reel location extraction, a travel marketplace, and **HostOS / StayOS** (a multi-tenant hotel PMS and AI guest messaging platform).

> **Project status:** The hosted production instances (`backend.travbuddy.in` and `api.hostos.in`) were retired in September 2026 (see [SHUTDOWN.md](./SHUTDOWN.md)). This repository contains the complete source code so you can self-host, run, or build on top of any part of the stack.

---

## Features

- **AI Trip Planner & TripDoc (`/api/v1/plan`, `/api/v3/trips`)**
  - Parallel orchestration of multi-leg transport (flights, trains, buses), day-by-day itineraries, and hotel recommendations via Vertex AI (Gemini).
  - Multi-stop routes (`origin → stops… → destination → origin`), budget-aware supervisor interventions, anti-hallucination confidence scoring, and Google Places enrichment.
  - **TripDoc v3 (`/api/v3/trips`)**: Single JSON trip document model with async background generation, instant selection updates, and partial itinerary/hotel re-search.
- **Live Travel Search & Recommendation Engine (`/api/v1/search`, `/api/v1/recommend`)**
  - Aggregates live inventory from Booking.com, Agoda, EaseMyTrip, Ixigo, GoToGate, goSTOPS, RedBus, FlixBus, and Indian Railways (ConfirmTKT).
  - Hybrid funnel: hard constraint filtering → deterministic L2 feature scoring → `pgvector` semantic re-ranking → optional streamed Gemini L3 re-rank & explanations.
- **Instagram Reel → Location Timeline (`/api/v1/reels`)**
  - Async Celery pipeline: `yt-dlp` / embed fallback download → `PySceneDetect` scene boundaries → `OpenCV` keyframe extraction → `pHash` deduplication → Gemini Vision location extraction → timeline merging → Supabase persistence.
  - Supports job polling and Server-Sent Events (SSE) progress streaming, with URL-level result caching.
- **Marketplace, Trips & Community (`/api/v1/marketplace`, `/api/v1/trips`, `/api/v1/groups`)**
  - Partner listing extraction from brochures/PDFs/text using Gemini, availability and pricing matching, Razorpay checkout and webhooks, saved trips/documents, and travel groups.
- **HostOS / StayOS — Hotel PMS & AI Guest Messaging (`ENABLE_HOTEL_OS=true`)**
  - Optional second deployment mode (`/api/v1/pms/*`, `/api/v1/messaging/*`) for property management (rooms, quotes, reservations, front-desk check-in, folios, hotel websites) and automated guest messaging over WhatsApp Cloud API, Instagram DMs, and web chat.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| **API Framework** | FastAPI (`uvicorn`), Pydantic v2, SlowAPI (rate limiting) |
| **Database & Auth** | Supabase (PostgreSQL + `pgvector` + JWT Auth with local JWKS/HS256 verification) |
| **AI & Embeddings** | Google Vertex AI (`google-genai` SDK — Gemini 2.5 / 3.x & `text-embedding-005`) |
| **Task Queue & Cache** | Celery + Redis (async jobs, scheduled tasks, session signals, rate limits) |
| **Video & Vision** | `yt-dlp`, `ffmpeg`, `scenedetect`, `opencv-python-headless`, `ImageHash` |
| **Enrichment & Storage** | Google Places API (`googlemaps` + `diskcache`), Google Cloud Storage (GCS) |
| **Payments & Email** | Razorpay (`razorpay`), Resend |
| **Observability** | `structlog` (correlated `X-Request-Id` tracing), Sentry (`sentry-sdk`) |

---

## Architecture Overview

The codebase supports running the **Marketplace / Trip Planner** mode by default, or **HostOS / StayOS** when `ENABLE_HOTEL_OS=true` (designed to point at separate Supabase projects).

```text
Client / Frontend
       │
       ▼
 FastAPI (app/main.py) ──► Supabase (Postgres + pgvector + Auth)
       │               ──► Vertex AI (Gemini + Embeddings) + Google Places
       │               ──► Live Travel Providers (Hotels / Flights / Trains / Buses)
       ▼
     Redis ◄──► Celery Worker (Reels pipeline, Reco tasks, StayOS messaging)
           ◄──► Celery Beat   (Nightly/weekly recommendation & maintenance jobs)
```

---

## Prerequisites

- **Python 3.11+** (or Docker + Docker Compose)
- **System dependencies** (for local non-Docker runs): `ffmpeg` and OpenCV shared libraries (`libglib2.0-0`, `libsm6`, `libxext6`, `libxrender-dev`, `libgomp1` on Debian/Ubuntu; `brew install ffmpeg` on macOS)
- **Redis 7+**
- **Supabase project** (PostgreSQL with `pgvector` enabled)
- **Google Cloud project** with Vertex AI API enabled (and optionally Places API + a GCS bucket for video/frame/media uploads)

---

## Quick Start (Local Development)

### 1. Clone and install dependencies

```bash
git clone <your-repo-url>
cd travbuddy_backend

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Fill in the required values in `.env` (see [Environment Variables](#environment-variables)). For local development, set:

```env
APP_ENV=development
EXPOSE_API_DOCS=true
REDIS_URL=redis://localhost:6379/0
```

> **Note:** `EXPOSE_API_DOCS` defaults to `false` for production safety. Set `EXPOSE_API_DOCS=true` locally to enable Swagger UI (`/docs`), ReDoc (`/redoc`), and `/openapi.json`.

### 3. Set up the Supabase database

1. Run [`schema.sql`](./schema.sql) in your Supabase SQL Editor to create the core tables and helper functions.
2. Apply migrations from [`migrations/`](./migrations/) as needed — in particular [`migrations/2026-07-13_reco_engine.sql`](./migrations/2026-07-13_reco_engine.sql) if you use the recommendation engine (`pgvector` tables and `match_hotels` RPC).

### 4. Start the services

You need Redis, the FastAPI server, and (for background jobs like Reels and recommendations) a Celery worker:

```bash
# Terminal 1: Redis (skip if running via Docker or Homebrew service)
redis-server

# Terminal 2: FastAPI server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 3: Celery worker (Reels pipeline + recommendation jobs)
celery -A app.workers.celery_app.celery_app worker --loglevel=info

# Terminal 4 (optional): Celery beat scheduler (run at most ONE instance)
celery -A app.workers.celery_app.celery_app beat --loglevel=info
```

Once running:
- **API root:** `http://localhost:8000/`
- **Health check:** `http://localhost:8000/api/v1/health`
- **Swagger UI:** `http://localhost:8000/docs` (when `EXPOSE_API_DOCS=true`)

---

## Running with Docker Compose

Two Compose definitions are included:

- [`docker-compose.yml`](./docker-compose.yml) — Marketplace / Trip Planner stack (`redis`, `api`, `worker`, `beat`)
- [`docker-compose.stayos.yml`](./docker-compose.stayos.yml) — HostOS / StayOS stack (`redis`, `api`, `worker` subscribed to `-Q hotelos`)

Before running `docker compose up -d --build` in your own environment:

1. Update the GCP service account volume mount in `docker-compose.yml` (it currently points to a host-specific path `/home/sahilverma/travbuddy_backend.json:/secrets/gcp_key.json:ro`) or remove the mount and pass `GOOGLE_SERVICE_ACCOUNT_JSON` via `.env`.
2. Update or remove the `traefik.http.routers.*.rule` host labels if you are not deploying behind Traefik on those domains.
3. Supply your environment variables (via `.env` or your orchestrator).

```bash
docker compose up -d --build
docker compose logs -f api
```

> **Important:** Never scale the `beat` service above 1 replica, or scheduled jobs will execute twice.

---

## Environment Variables

All settings are defined in [`app/config.py`](./app/config.py) and templated in [`.env.example`](./.env.example).

### Core (Required for main API)

| Variable | Description |
| --- | --- |
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_SERVICE_KEY` | Supabase `service_role` key (server-side DB access) |
| `SUPABASE_ANON_KEY` | Supabase `anon` key (used for auth / JWKS) |
| `GCP_PROJECT` | Google Cloud project ID for Vertex AI (Gemini + embeddings) |
| `GCP_LOCATION` | Primary Vertex AI region (default: `us-central1`) |
| `GOOGLE_APPLICATION_CREDENTIALS` | Path to GCP service account JSON key (or use `GOOGLE_SERVICE_ACCOUNT_JSON` / ADC) |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | Full GCP service account JSON as a single-line string (used by Vertex / GCS) |
| `REDIS_URL` | Redis connection URL (default: `redis://localhost:6379/0`) |

### Optional & Feature-Specific

| Variable | Description |
| --- | --- |
| `SUPABASE_JWT_SECRET` | Enables local HS256 JWT verification without a Supabase network round-trip |
| `GOOGLE_MAPS_API_KEY` | Google Places API key for place ratings, photos, hours, and Maps URLs |
| `GCS_BUCKET_NAME` | GCS bucket for Reel video/frame uploads and hotel media |
| `INSTAGRAM_PROXY_URL` | Optional HTTP rotating proxy (`http://user:pass@host:port/`) for Reel downloads & travel scrapers |
| `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `RAZORPAY_WEBHOOK_SECRET` | Razorpay credentials for marketplace & stay payments |
| `RESEND_API_KEY`, `RESEND_FROM`, `SITE_URL` | Transactional email sending via Resend |
| `ADMIN_API_PASSCODE` | Shared secret (`X-Admin-Passcode`) for `/api/v1/admin/*` routes |
| `BULK_EXPLORE_API_KEY` | Shared secret for bulk explore generation |
| `SENTRY_DSN` | Sentry error & performance tracking (leave empty locally to disable) |
| `EXPOSE_API_DOCS` | Set `true` to serve `/docs`, `/redoc`, and `/openapi.json` (default: `false`) |
| `ALLOWED_ORIGINS` | Comma-separated CORS origins (`*` allowed in non-production) |

### HostOS / StayOS Mode (Optional)

Set `ENABLE_HOTEL_OS=true` to mount `/api/v1/pms/*`, `/api/v1/messaging/*`, and (optionally) `/api/v1/google/*`. See [`DEPLOYING_STAYOS.md`](./DEPLOYING_STAYOS.md) for full details.

| Variable | Description |
| --- | --- |
| `ENABLE_HOTEL_OS` | Mounts PMS & messaging routes (default: `false`) |
| `MESSAGING_CREDENTIALS_KEY` | Fernet key used to encrypt per-hotel channel tokens at rest |
| `WHATSAPP_APP_ID`, `WHATSAPP_APP_SECRET`, `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_VERIFY_TOKEN`, `WHATSAPP_CONFIG_ID` | Meta WhatsApp Cloud API & Embedded Signup credentials |
| `INSTAGRAM_APP_ID`, `INSTAGRAM_OAUTH_CLIENT_SECRET`, `INSTAGRAM_APP_SECRET`, `INSTAGRAM_VERIFY_TOKEN`, `INSTAGRAM_REDIRECT_URI` | Meta Instagram Login & Messaging webhooks |
| `ENABLE_GOOGLE_BUSINESS_PROFILE`, `GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET` | Optional Google Business Profile integration |

---

## Recommendation Engine Bootstrap

If you want to populate embeddings, personality tags, and SEO profiles for hotels in Supabase:

```bash
# 1. Verify required tables, columns, and pgvector RPC exist
python scripts/reco_bootstrap.py --check

# 2. Seed the 24 personality centroids (one-time setup)
python scripts/reco_bootstrap.py --seed-centroids

# 3. Sanity-check backfill on a single destination
python scripts/reco_bootstrap.py --backfill --seo --limit 30 --destination Udaipur

# 4. Run full backfill (resumable via doc-hash)
python scripts/reco_bootstrap.py --backfill --seo
```

See [`docs/recommendation_engine.md`](./docs/recommendation_engine.md) for the full ranking architecture.

---

## Key API Endpoints

Most user-facing endpoints expect a Supabase JWT in `Authorization: Bearer <token>`.

| Endpoint | Description |
| --- | --- |
| `GET /api/v1/health` | Dependency health check (Redis, Celery worker, Supabase, Vertex, Places, `yt-dlp`) |
| `POST /api/v1/plan` | Unified AI trip planner (transport + itinerary + hotels + budget; rate-limited `10/hour`) |
| `POST /api/v3/trips` | Create a TripDoc and start async generation |
| `GET /api/v3/trips/{trip_id}` | Poll TripDoc state and rendered plan |
| `PATCH /api/v3/trips/{trip_id}/select` | Persist user selections (transport/hotel) without re-running AI |
| `POST /api/v1/search/{flights,hotels,buses,trains,all}` | Live multi-provider travel search & smart search variants |
| `POST /api/v1/recommend/{hotels,transport,journey,explain,feedback}` | Personalized recommendation, ranking explanations, and feedback logging |
| `POST /api/v1/reels/analyze` | Queue an Instagram Reel location extraction job (`202 Accepted`) |
| `GET /api/v1/reels/jobs/{job_id}` | Poll Reel job status |
| `GET /api/v1/reels/jobs/{job_id}/result` | Fetch completed Reel location timeline |
| `GET /api/v1/reels/jobs/{job_id}/stream` | Stream real-time Reel pipeline progress via SSE |
| `GET/POST /api/v1/trips/{trip_id}/*` | Saved reels, bookings, files, and trip documents |
| `GET/POST /api/v1/groups/*` | Travel groups and community trips |
| `POST /api/v1/marketplace/*` | Partner listing extraction, inventory matching, and Razorpay payments |
| `/api/v1/pms/*`, `/api/v1/messaging/*` | HostOS PMS and WhatsApp/Instagram webhooks (when `ENABLE_HOTEL_OS=true`) |

### Example: Plan a Trip (`POST /api/v1/plan`)

```bash
curl -X POST http://localhost:8000/api/v1/plan \
  -H "Authorization: Bearer <SUPABASE_JWT>" \
  -H "Content-Type: application/json" \
  -d '{
    "origin": "Delhi",
    "destination": "Goa",
    "departure_date": "2027-02-14",
    "days": 5,
    "travelers": 2,
    "budget": "mid",
    "interests": ["beach", "food"],
    "stops": []
  }'
```

### Example: Analyze an Instagram Reel (`POST /api/v1/reels/analyze`)

```bash
curl -X POST http://localhost:8000/api/v1/reels/analyze \
  -H "Authorization: Bearer <SUPABASE_JWT>" \
  -H "Content-Type: application/json" \
  -d '{"reel_url": "https://www.instagram.com/reel/<shortcode>/"}'
```

---

## Project Structure

```text
travbuddy_backend/
├── app/
│   ├── main.py              # FastAPI app, lifespan, CORS, request correlation, error handlers
│   ├── config.py            # Pydantic settings & feature flags
│   ├── agents/              # Supervisor, route, budget, and recommendation agents
│   ├── api/
│   │   ├── v1/              # plan_trip, search, recommend, reels, trips, groups, marketplace, pms, messaging, admin
│   │   └── v3/              # TripDoc CRUD endpoints
│   ├── core/                # Circuit breakers, rate limiter, async HTTP, SSE, timeout middleware
│   ├── db/                  # Supabase client & resilient DB executor
│   ├── deps/                # Auth, admin, B2B, and property membership dependencies
│   ├── models/              # Shared Pydantic request/response models
│   ├── pipeline/            # Reel downloader, PySceneDetect, OpenCV frames, pHash dedup, Gemini vision
│   ├── reco/                # Intent extraction, profile merge, features, pgvector store, ranking
│   ├── services/            # Travel provider integrations, Vertex AI, Places, PMS, messaging, Razorpay
│   ├── storage/             # GCS and Supabase persistence helpers
│   ├── trips_doc/           # TripDoc generation & serialization
│   └── workers/             # Celery app, job store, and background/scheduled tasks
├── docs/                    # Architecture & engineering design docs
├── migrations/              # Incremental Supabase SQL migrations
├── scripts/                 # Bootstrap, smoke tests, scrapers, and E2E verification scripts
├── tests/                   # Pytest unit and integration test suite
├── Dockerfile
├── docker-compose.yml
├── docker-compose.stayos.yml
├── requirements.txt
└── schema.sql
```

---

## Testing & Diagnostics

Run the test suite with `pytest`:

```bash
pytest -q
```

Run targeted suites or diagnostics:

```bash
# Recommendation engine & scoring tests
pytest tests/test_reco_engine.py tests/test_scoring.py tests/test_hotel_dedup.py -v

# Core endpoint smoke test (against a running server)
python scripts/smoke_test.py

# Dump consolidated scraper responses for debugging
HOTEL_CITY=Jaipur ORIGIN=Mumbai DEST=Goa python scripts/dump_consolidated.py
```

---

## Additional Documentation

- [`product.md`](./product.md) — product scope, intended users, and project status
- [`Features.md`](./Features.md) — backend capability and API catalogue
- [`technicals.md`](./technicals.md) — architecture and local development notes
- [`integration.md`](./integration.md) — external services and configuration
- [`CLAUDE.md`](./CLAUDE.md) — repository guide for coding agents
- [`docs/features/index.md`](./docs/features/index.md) — detailed feature guides by product area
- [`docs/api_surface.md`](./docs/api_surface.md) — route-family reference, including optional HostOS APIs
- [`docs/database_schema.md`](./docs/database_schema.md) — database entities, relationships, and RLS overview
- [`docs/database_migrations.md`](./docs/database_migrations.md) — migration index and project cautions
- [`docs/workers_and_schedules.md`](./docs/workers_and_schedules.md) — Celery tasks, queues, and beat schedules
- [`docs/security_and_tenancy.md`](./docs/security_and_tenancy.md) — authentication, authorization, and tenant boundaries
- [`docs/codebase_map.md`](./docs/codebase_map.md) — source packages, tests, and scripts
- [`readme_run.md`](./readme_run.md) — Quick command cheat sheet for local dev and backfill jobs
- [`DEPLOYING_STAYOS.md`](./DEPLOYING_STAYOS.md) — Dual-deployment architecture for HostOS / StayOS
- [`SHUTDOWN.md`](./SHUTDOWN.md) — Notes on retired production infrastructure
- [`docs/recommendation_engine.md`](./docs/recommendation_engine.md) — Hybrid recommendation pipeline design
- [`docs/trip_engine_architecture.md`](./docs/trip_engine_architecture.md) — Multi-stop trip engine architecture
- [`docs/trip_graph.md`](./docs/trip_graph.md) — Trip graph data model
- [`docs/persistence_architecture.md`](./docs/persistence_architecture.md) — Persistence & caching architecture
- [`docs/agent_architecture_map.md`](./docs/agent_architecture_map.md) — Agent runtime & toolsets map

