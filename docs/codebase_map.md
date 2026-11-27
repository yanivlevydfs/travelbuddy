# Codebase Map, Tests, and Scripts

## Application packages

| Path | Responsibility |
| --- | --- |
| `app/api/v1/` | Version 1 routes for planning, search, recommendations, groups, marketplace, Reels, travellers, and admin |
| `app/api/v3/` | TripDoc v3 routes |
| `app/agents/` | Trip route, budget, recommendation, and supervisor agents |
| `app/trips_doc/`, `app/trip_state/` | Trip document generation, state, selection, and persistence |
| `app/reco/` | Recommendation intent, profiles, features, ranking, feedback, and vector store |
| `app/pipeline/` | Reel download, scene detection, frame extraction, deduplication, and vision |
| `app/services/` | Provider adapters, business services, PMS, messaging, AI, payments, and integrations |
| `app/db/`, `app/storage/` | Supabase clients and GCS/Supabase storage helpers |
| `app/deps/` | Authentication, admin, B2B agency, and property membership dependencies |
| `app/core/` | HTTP, background helpers, generation queue, limits, SSE, timeouts, and metrics |
| `app/workers/` | Celery app, Reel/recommendation/messaging/marketing/partner tasks |

`app/main.py` creates the FastAPI application. `app/config.py` defines
environment-backed settings and feature flags.

## Existing design documentation

The `docs/` directory contains focused engineering designs, including:

- Trip engine, trip graph, and TripState removal/simplification plans.
- Recommendation engine and persistence architecture.
- Agent architecture and agent arbitration.
- Production readiness, partner stays, and growth-operations plans.

Treat design/plan documents as design context, not necessarily as proof that
every proposed phase is deployed. Confirm behavior against the code and current
schema.

## Test areas

Tests are in `tests/`. Representative groups include:

- Planner and TripDoc: `test_long_journeys.py`, `test_trip_doc.py`,
  `test_builder.py`, `test_compose_actions.py`.
- Search and recommendations: `test_reco_engine.py`, `test_scoring.py`,
  `test_hotel_dedup.py`, `test_hotel_filters.py`.
- Reel pipeline: `test_downloader.py`, `test_scene_detector.py`,
  `test_pipeline.py`.
- Marketplace and payments: `test_marketplace_*.py`, `test_partner_deals.py`,
  `test_stay_payment.py`, `test_billing.py`.
- HostOS and AI employees: `test_ai_runtime.py`, `test_ai_tools.py`,
  `test_hotel_os_routes.py`, `test_pms_*.py`, `test_negotiation_end_to_end.py`.
- Messaging: `test_inbound_race.py`, `test_turn_idempotency.py`,
  `test_instagram_oauth.py`, `test_webhook_retry.py`, `test_channel_capability.py`.
- Access and invariants: `test_auth_clock_skew.py`,
  `test_tenant_isolation.py`, `test_upload_sniffing.py`,
  `test_inventory_hygiene.py`.

Run focused tests first, then the suite:

```powershell
.\.venv\Scripts\python.exe -m pytest tests\test_reco_engine.py -q
.\.venv\Scripts\python.exe -m pytest -q
```

Some evaluator or end-to-end tests require configured Supabase, Redis, Vertex,
or provider credentials. Inspect fixtures and test setup before running a test
that may reach external services or mutate data.

## Scripts

The `scripts/` directory contains:

- **Setup and data:** `reco_bootstrap.py`, `scrape_gostops.py`,
  `seed_group_trips.py`, `seed_tenant.py`, `migrate_partners.py`.
- **Inventory maintenance:** `reconcile_inventory.py`,
  `repair_inventory.py`, `backup_hotel_os.py`.
- **Health/configuration:** `smoke_test.py`, `prod_smoke.py`,
  `stayos_backend_check.py`, `meta_config_check.py`, `token_audit.py`.
- **Journey and integration checks:** `*_e2e.py`, `meta_smoke.py`,
  `production_whatsapp_check.py`, `vertex_e2e.py`, and tenant-isolation probes.

Scripts are not all read-only. Before running one, inspect its arguments and
environment requirements; it may call live APIs, mutate a Supabase database,
send messages, create payments, or change inventory. Use disposable test data
and non-production credentials unless the script explicitly documents
otherwise.
