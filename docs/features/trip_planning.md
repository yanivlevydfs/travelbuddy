# Trip Planning and TripDoc

## Capabilities

### Unified AI planner

`POST /api/v1/plan` runs the multi-agent planning flow. It accepts a trip
request and coordinates route planning, transport search, itinerary generation,
hotel suggestions, and budget supervision. The planner supports multi-stop
journeys, date/day constraints, and selections among transport modes.

Planning components live primarily in `app/api/v1/plan_trip/` and
`app/agents/`:

- **Route agent:** proposes route legs and transport modes.
- **Budget agent:** monitors cost and can influence mode choices when the
  current plan exceeds the requested budget.
- **Recommendation agent:** supports the planning/recommendation workflow.
- **Supervisor:** coordinates agent phases and the overall plan.
- **Places enrichment:** can add Google Places details when configured.

Gemini runs through Vertex AI. The application owns validation and business
decisions; model output is not treated as verified inventory, final prices, or
booking state.

### TripDoc v3

TripDoc is the newer persisted trip document API. It supports asynchronous
generation, state retrieval, itinerary edits, place search, hotel re-search,
and selecting options without regenerating the entire trip.

Key routes:

- `POST /api/v3/trips` — create a trip and initiate its workflow.
- `GET /api/v3/trips/{trip_id}` — read the trip document and current state.
- `POST /api/v3/trips/{trip_id}/actions` — request a supported trip action.
- `PATCH /api/v3/trips/{trip_id}/itinerary` — edit itinerary content.
- `GET /api/v3/trips/{trip_id}/places` — search places in trip context.
- `POST /api/v3/trips/{trip_id}/search-hotels` — re-search hotels.
- `PATCH /api/v3/trips/{trip_id}/select` — persist an option selection.

The implementation is in `app/api/v3/`, `app/trips_doc/`, and `app/trip_state/`.
Persistence and search-result separation are covered in
[Persistence architecture](../persistence_architecture.md).

## Saved traveller data

The v1 traveller APIs include saved trips, trip itinerary updates, journey
views, bookings, documents, files, and saved Reel attachments. These are
separate from TripDoc v3 and continue to use their existing v1 routes.

Relevant route families:

- `/api/v1/users/trips*`
- `/api/v1/trips/{trip_id}/bookings*`
- `/api/v1/trips/{trip_id}/documents*`
- `/api/v1/trips/{trip_id}/files/upload`
- `/api/v1/trips/{trip_id}/reels*`
- `/api/v1/trips/{trip_id}/saved-reels*`

## Dependencies and limitations

- Supabase is required for persistence and authenticated user data.
- Vertex AI credentials are needed for Gemini-backed planning.
- Search results depend on the configured travel providers.
- This repository contains the backend; the consuming traveller UI is separate.

For request schemas, use the local OpenAPI UI with `EXPOSE_API_DOCS=true`.
