# Platform, Discovery, Admin, and Observability Features

## Discovery and destination catalog

- Autocomplete airports and destinations.
- Browse hotel cities, hotel details, and hostel listings/details/availability.
- Discover vibe trips and weekend plans.
- View live/recent community activity.

Key routes: `/api/v1/autocomplete/*`, `/api/v1/hotels*`,
`/api/v1/hostels*`, `/api/v1/discover/*`, and `/api/v1/community/*`.

## Onboarding and user profile

- `POST /api/v1/onboarding/generate-feed` creates user feed content.
- `/api/v1/users/*` returns user statistics and saved trips, and supports trip
  reads, updates, and deletion.
- Referral APIs provide referral status and redemption.

These features depend on Supabase user identity and the relevant schema.

## Admin operations

Admin route families cover:

- Trip and listing review, validation, publish/unpublish, and editing.
- Partner invitations, actions, terms, and login links.
- Hotel billing audit/ledger, payout operations, coupon management, and credit
  adjustments.
- Admin audit events, queue inspection, and review operations.
- AI margin and catalog management.

Admin routes require `ADMIN_API_PASSCODE` via the `X-Admin-Passcode` header.
Without a configured passcode, the routes fail closed. The optional
`X-Admin-Actor` value is for audit attribution, not an identity boundary.

## Metrics and health

- `GET /api/v1/health` reports Redis, worker, Supabase, Vertex/Places, media
  tooling, and channel-credential readiness.
- `/api/v1/metrics/providers`, `/scrapers`, and `/dashboard` expose service
  health and provider metrics.
- `POST /api/v1/metrics/scrapers/reset` resets scraper metrics.

## API documentation

Swagger, ReDoc, and the OpenAPI schema are disabled by default. Set
`EXPOSE_API_DOCS=true` in local development to enable `/docs`, `/redoc`, and
`/openapi.json`. The [API surface reference](../api_surface.md) lists route
families.
