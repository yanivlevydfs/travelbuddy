# API Surface Reference

This is a route-family index, not a replacement for request/response schemas.
For exact models and parameters, run the API locally with
`EXPOSE_API_DOCS=true` and open `/docs` or `/openapi.json`.

All v1 routes use the `/api/v1` prefix. TripDoc v3 routes use `/api/v3`.
Authentication is route-specific: traveller and operator data generally
requires a Supabase bearer token, while selected catalog, public booking, and
webhook routes have different access rules.

## Default travel deployment

### Health and operations

| Route family | Purpose |
| --- | --- |
| `GET /` | Service name, version, and documentation/health links |
| `GET /api/v1/health` | Dependency readiness for Redis, worker, Supabase, Google services, and media tooling |
| `/api/v1/metrics/*` | Provider dashboard, provider health, scraper metrics, and metric reset |

### Planning and recommendations

| Route family | Purpose |
| --- | --- |
| `POST /api/v1/plan` | AI trip plan generation |
| `/api/v3/trips*` | TripDoc create/read/actions, itinerary edits, place search, hotel search, and selection |
| `/api/v1/recommend/*` | Hotel, transport, journey recommendations, explanations, and feedback |

### Search and discovery

| Route family | Purpose |
| --- | --- |
| `/api/v1/search/*` | Combined, flight, hotel, bus, and train searches; smart search variants and hotel filters |
| `/api/v1/autocomplete/*` | Airport and destination suggestions |
| `/api/v1/hotels*` | Hotel catalog, hotel details, and cities |
| `/api/v1/hostels*` | Hostel catalog, details, availability, and live status |
| `/api/v1/discover/*` | Vibe trips and weekend plans |
| `/api/v1/community/*` | Live and recent community activity |

### Traveller trips and media

| Route family | Purpose |
| --- | --- |
| `/api/v1/users/*` | User statistics and saved trip read/update/delete operations |
| `/api/v1/trips/{trip_id}/bookings*` | Trip booking records, confirmation files, and upload initialization |
| `/api/v1/trips/{trip_id}/documents*` | Document create, upload initialization, confirmation, list, and delete |
| `/api/v1/trips/{trip_id}/files/upload` | Trip file upload |
| `/api/v1/trips/{trip_id}/reels*` | Trip Reel attachments |
| `/api/v1/reels/*` | Analyze, poll, stream, retrieve, save, and like Reel results |
| `GET /api/v1/images/{photo_reference}` | Rate-limited Google Places photo proxy; hidden from OpenAPI because it is an image response |

### Marketplace, groups, and referrals

| Route family | Purpose |
| --- | --- |
| `/api/v1/marketplace/extract/*` | Extract partner listings from file or text |
| `/api/v1/marketplace/match`, `/places`, `/quote` | Inventory match, place lookup, and pricing quote |
| `/api/v1/marketplace/checkout*`, `/bookings/*`, `/webhook` | Checkout, verification, booking lookup, and Razorpay webhook |
| `/api/v1/marketplace/listings/*` | Submit, delete, publish/unpublish partner listings |
| `/api/v1/marketplace/trips/*` | Submit/unpublish partner trips and update trip/departure seats |
| `/api/v1/marketplace/subscribe*` | Start, verify, cancel, or apply coupon to partner subscription |
| `/api/v1/groups*` | Create/join/preview groups, messages, AI assistance, polls, and expenses |
| `/api/v1/referral/*` | Referral status and code redemption |

### Admin and onboarding

| Route family | Purpose |
| --- | --- |
| `/api/v1/admin/*` | Admin review, trips, listings, partners, billing, coupons, credits, payouts, audit, and queue |
| `POST /api/v1/onboarding/generate-feed` | Generate onboarding/home-feed content |

Admin calls require `X-Admin-Passcode`; see
[Security and tenancy](./security_and_tenancy.md).

## Optional HostOS / StayOS deployment

These routes are mounted only when `ENABLE_HOTEL_OS=true`, and the service must
use the HostOS Supabase project:

| Route family | Purpose |
| --- | --- |
| `/api/v1/pms/me`, `/properties*` | Property setup, inventory, rates, reservations, check-in/out, folios, tasks, analytics, billing, agents, and hotel website |
| `/api/v1/pms/public/*` | Public hotel discovery, availability, quotes, booking, check-in, and payment confirmation |
| `/api/v1/pms/agency/*` | Contracted agency availability, quotes, and bookings |
| `/api/v1/messaging/whatsapp/*` | WhatsApp webhook and property onboarding/connection operations |
| `/api/v1/messaging/instagram/*` | Instagram webhook, OAuth callback, connection, deauthorization, and data-deletion operations |
| `/api/v1/messaging/properties/*` | Per-property channel administration and tests |
| `/api/v1/google/*` | Optional Google Business Profile OAuth, locations, reviews, and posts |

Google Business Profile routes additionally require
`ENABLE_GOOGLE_BUSINESS_PROFILE=true`. The HostOS overview is in
[hostos_pms.md](./features/hostos_pms.md).

## Diagnostic route

The application also registers `/sentry-debug` for Sentry error-path testing.
Treat it as an operational diagnostic, not a user-facing feature; keep API docs
and diagnostic surfaces disabled or appropriately restricted in production.
