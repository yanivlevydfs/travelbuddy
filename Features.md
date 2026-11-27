# TravBuddy Feature Catalogue

This catalogue describes the backend capabilities in the repository. Routes can
depend on feature flags, external credentials, database schema, Redis, or
background workers.

## Detailed feature guides

- [Feature guide index](./docs/features/index.md)
- [Trip planning and TripDoc](./docs/features/trip_planning.md)
- [Search and recommendations](./docs/features/search_and_recommendations.md)
- [Reel intelligence](./docs/features/reels.md)
- [Marketplace and community](./docs/features/marketplace_and_community.md)
- [HostOS / StayOS PMS](./docs/features/hostos_pms.md)
- [Messaging and marketing](./docs/features/messaging_and_marketing.md)
- [Platform and admin](./docs/features/platform_and_admin.md)

For detailed reference material, see the [API surface](./docs/api_surface.md),
[database schema](./docs/database_schema.md), [workers and schedules](./docs/workers_and_schedules.md),
[security and tenancy](./docs/security_and_tenancy.md), and
[codebase map](./docs/codebase_map.md). The [database column catalog](./docs/database_columns.md)
lists checked-in table definitions and distinguishes the effective TripDoc row from its historical DDL.

## Trip planning and TripDoc

- Multi-city planning with transport, hotel recommendations, budget-aware
  decisions, and day-wise itinerary content.
- Vertex AI / Gemini orchestration, confidence scoring, and Google Places
  enrichment.
- TripDoc v3 creates a durable trip document and supports polling, selection
  changes, and partial re-search.

Key routes:

- `POST /api/v1/plan`
- `POST /api/v3/trips`
- `GET /api/v3/trips/{trip_id}`
- `PATCH /api/v3/trips/{trip_id}/select`

## Live search and recommendations

- Search across supported hotel, flight, train, and bus providers.
- Apply hard constraints and deterministic scoring before optional semantic
  reranking and Gemini explanations.
- Capture recommendation feedback and support background recommendation work.

Key route groups:

- `POST /api/v1/search/{flights,hotels,buses,trains,all}`
- `POST /api/v1/recommend/{hotels,transport,journey,explain,feedback}`

Provider coverage is listed in [integration.md](./integration.md).

## Reel-to-location intelligence

- Accept an Instagram Reel URL and process it asynchronously.
- Download or extract video, detect scenes, sample and deduplicate frames, then
  use Gemini Vision to identify locations.
- Enrich recognized places when Google Places is configured.
- Poll job status or stream progress using Server-Sent Events.

Key route group: `/api/v1/reels`

## Marketplace, trips, and groups

- Extract partner listings from supplied brochures, PDFs, or text.
- Match offers with inventory and pricing.
- Support marketplace checkout and payment webhooks.
- Provide saved-trip/document and group-trip APIs.

Key route groups: `/api/v1/marketplace`, `/api/v1/trips`, and
`/api/v1/groups`.

## HostOS / StayOS

Optional hotel operations APIs cover property setup, rooms, quotes,
reservations, front-desk check-in, folios, hotel websites, and messaging
workflows. WhatsApp and Instagram channel support depends on Meta credentials
and configuration.

Enable with `ENABLE_HOTEL_OS=true`. Key route groups:
`/api/v1/pms/*` and `/api/v1/messaging/*`.

## Operations and discovery

- `GET /api/v1/health` reports API dependencies and integration readiness.
- `/docs`, `/redoc`, and `/openapi.json` are disabled by default; enable them
  for local development with `EXPOSE_API_DOCS=true`.
- Admin routes are under `/api/v1/admin/*` and require their configured
  passcode.

For service setup and credentials, see [integration.md](./integration.md).
