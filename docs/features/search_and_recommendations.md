# Travel Search and Recommendations

## Search APIs

The v1 search surface exposes:

- `POST /api/v1/search/all`
- `POST /api/v1/search/flights` and `/flights/smart`
- `POST /api/v1/search/hotels` and `/hotels/smart`
- `GET /api/v1/search/hotels/filters`
- `POST /api/v1/search/buses`
- `POST /api/v1/search/trains`
- `GET /api/v1/autocomplete/airports`
- `GET /api/v1/autocomplete/destinations`

There are also traveller catalog APIs for hotel and hostel content:
`/api/v1/hotels*` and `/api/v1/hostels*`.

Provider adapters are organized under `app/services/`, including flights,
hotels, buses, trains, goSTOPS, Agoda, EaseMyTrip, GoToGate, Ixigo, Booking.com,
RedBus, FlixBus, and ConfirmTKT. Inventory and availability are provider
dependent; a route being present does not guarantee every provider is reachable
or returning results.

## Recommendation API

- `POST /api/v1/recommend/hotels`
- `POST /api/v1/recommend/transport`
- `POST /api/v1/recommend/journey`
- `POST /api/v1/recommend/explain`
- `POST /api/v1/recommend/feedback`

The recommendation modules in `app/reco/` handle intent, user profile,
personality signals, features, ranking, explanations, feedback, and vector
retrieval.

## Recommendation pipeline

1. Parse the request and infer relevant preferences or intent.
2. Apply hard constraints such as destination, dates, party size, and budget.
3. Score eligible candidates using deterministic features.
4. Optionally rerank semantically using Supabase `pgvector`.
5. Optionally ask Gemini to explain or further rerank results.
6. Store feedback and use scheduled refresh tasks for selected aggregates.

The exact scoring features and vector schema are documented in
[recommendation_engine.md](../recommendation_engine.md). The vector migration
is `migrations/2026-07-13_reco_engine.sql`; the search-result store is defined
in `migrations/2026-07-14_search_store.sql`.

## Related features

- Trip planning can invoke the same travel search and recommendation services.
- Partner stays can be incorporated into travel search; see
  [partner stays in search](../partner_stays_in_search.md).
- Search/provider health is reported by `/api/v1/metrics/providers` and
  `/api/v1/metrics/scrapers`.

## Dependencies

Provider HTTP access is required for live results. Recommendation persistence
uses Supabase; semantic reranking requires the relevant database migration and
pgvector RPC. Gemini explanations and embeddings require Vertex AI credentials.
