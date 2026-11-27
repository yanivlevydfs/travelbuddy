# Integration Guide

TravBuddy depends on external services for persistence, AI, search, media,
payments, and messaging. Configure credentials in a local `.env` or a secret
manager; do not commit credentials or expose service-role keys to clients.
`.env.example` and `app/config.py` are the configuration references.

## Core services

| Integration | Purpose | Configuration / notes |
| --- | --- | --- |
| Supabase | PostgreSQL, Auth/JWT verification, and application data | `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `SUPABASE_ANON_KEY`; `SUPABASE_JWT_SECRET` enables local HS256 verification. Keep the service key server-side. |
| Vertex AI / Gemini | Trip planning, extraction, recommendation explanations, and video vision | Configure `GCP_PROJECT`, `GCP_LOCATION`, and Google credentials using ADC, `GOOGLE_APPLICATION_CREDENTIALS`, or `GOOGLE_SERVICE_ACCOUNT_JSON`. |
| Redis | Celery broker and coordination/cache features | `REDIS_URL`, defaulting to `redis://localhost:6379/0`. Run Redis for worker-backed features. |
| Google Places | Place enrichment | Optional `GOOGLE_MAPS_API_KEY`; without it, enrichment is unavailable. |
| Google Cloud Storage | Optional video, frame, and hotel-media storage | Configure `GCS_BUCKET_NAME` and Google credentials. |

## Travel inventory providers

The search layer contains integrations for Booking.com, Agoda, EaseMyTrip,
Ixigo, GoToGate, goSTOPS, RedBus, FlixBus, and Indian Railways through
ConfirmTKT. Provider availability and required access vary. Some providers use
public endpoints or browser-like HTTP clients; others may require provider
credentials, affiliate configuration, or proxy fallback.

Relevant settings include `BOOKING_AFFILIATE_ID`, ConfirmTKT settings, and
`INSTAGRAM_PROXY_URL` for the shared optional proxy fallback. Consult the
provider adapter in `app/services/` before enabling or changing an integration.

## Payments and email

- **Razorpay:** marketplace and StayOS payments; configure
  `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, and `RAZORPAY_WEBHOOK_SECRET`.
  Point the provider webhook to `/api/v1/marketplace/webhook` and subscribe to
  the events documented in `.env.example`.
- **Resend:** transactional email; configure `RESEND_API_KEY` and, if needed,
  `RESEND_FROM` and `SITE_URL`.

## Hotel messaging and identity

HostOS is disabled unless `ENABLE_HOTEL_OS=true`. It is designed to use its own
Supabase project and environment configuration.

- **WhatsApp Cloud API:** configure the `WHATSAPP_*` credentials and
  `MESSAGING_CREDENTIALS_KEY`.
- **Instagram messaging:** configure the `INSTAGRAM_*` application and webhook
  settings and `MESSAGING_CREDENTIALS_KEY`.
- **Google Business Profile:** optional; controlled by
  `ENABLE_GOOGLE_BUSINESS_PROFILE` and Google OAuth settings.

Webhook URLs, verification steps, and the complete deployment process are in
[DEPLOYING_STAYOS.md](./DEPLOYING_STAYOS.md).
For request authorization and tenant boundaries, see
[Security and tenancy](./docs/security_and_tenancy.md).

## Observability and operational settings

- `SENTRY_DSN` enables Sentry; leave it unset locally to disable reporting.
- `EXPOSE_API_DOCS=true` enables Swagger, ReDoc, and OpenAPI endpoints for
  local development. Keep API docs disabled in production unless intentionally
  protected and exposed.
- `ALLOWED_ORIGINS` controls browser CORS origins.
- `ADMIN_API_PASSCODE` protects admin routes; use a strong secret.

## Setup checklist

1. Choose the correct Supabase project for the deployment mode. Do not mix the
   travel and HostOS schemas.
2. Apply `schema.sql` and the required migrations to that project.
3. Configure only the service credentials needed for the features being run.
4. Start Redis before launching Celery workers or features that depend on it.
5. Check `GET /api/v1/health` and verify each required dependency is ready.
6. Never put service keys, webhook secrets, or cloud credentials in source
   control, client-side code, or issue/chat messages.
