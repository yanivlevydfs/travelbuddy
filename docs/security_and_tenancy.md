# Security and Tenant Isolation

This page maps the main security boundaries visible in the application and
schema. It is not a substitute for reviewing the endpoint, SQL policy, and
provider documentation involved in a specific change.

## Authentication

`app/deps/auth.py` verifies Supabase bearer tokens:

1. HS256 can be verified locally when `SUPABASE_JWT_SECRET` is configured.
2. RS256/ES256 tokens can be verified against the configured project's JWKS.
3. If local verification is unavailable for a token, the dependency can fall
   back to Supabase Auth.

`require_user` is the authenticated-user dependency. `optional_user` is only
for endpoints that allow anonymous access while attributing a valid caller; it
must not be used as an authorization check. Tokens are cached for a bounded
period and no longer than their own expiry.

The project URL is part of the trust boundary: the travel and HostOS Supabase
projects have separate auth keys and schemas. One process uses one project.

## Traveller data

The core schema uses Supabase `auth.uid()` policies for user-owned profiles,
trips, bookings, documents, Reels, folders, and other private data. Group
policies use membership/admin helper functions. TripState sharing permits
read-only access when visibility is `link` or `public`; owner operations remain
scoped to the authenticated user.

Several backend operations use the service-role client, which bypasses RLS.
Those routes must still verify ownership and authorization in application code.
Do not treat a service-role query as proof that the caller may access the row.

## HostOS property tenancy

`app/deps/property.py` resolves the property from the path and the caller's
verified identity. The caller's partner/member relationship is looked up on the
server; clients do not choose a partner identity in request data. A property
outside the caller's membership scope returns not-found rather than confirming
that another tenant's property exists.

Role permissions are checked per property and include owner, manager, accounts,
receptionist, housekeeping, maintenance, concierge, restaurant, and driver
roles. Extra member permissions may add grants; the database permission
function/RLS policy remains an important second boundary.

Travel agencies use a separate dependency in `app/deps/b2b.py`. The agency is
resolved from the authenticated user, and property contracts are checked by
the B2B service.

## Admin API

Admin routes require `ADMIN_API_PASSCODE` in the `X-Admin-Passcode` header.
The comparison is constant-time. If the passcode is unset, requests fail
closed; a wrong passcode does not confirm the admin route. `X-Admin-Actor` is
optional audit metadata and is self-reported, not an identity credential.
Transmit the passcode only over HTTPS.

## AI tool authorization and business truth

HostOS AI tools pass through a registry/runtime that validates arguments,
checks channel and role permissions, applies business prechecks and autonomy
rules, and records tool actions. Money, inventory, reservation, and payment
facts must come from the corresponding backend service/provider result, not
from generated prose. See [Agent arbitration](./agent_arbitration.md) and
[Agent architecture map](./agent_architecture_map.md).

## Webhooks and stored credentials

- WhatsApp and Instagram webhook handlers verify Meta signatures before
  processing inbound payloads.
- Per-property channel credentials are encrypted using
  `MESSAGING_CREDENTIALS_KEY`; configure a valid Fernet key before enabling
  channel connections.
- Razorpay payment callbacks must be verified with the configured webhook
  secret before changing application payment state.
- Service keys, OAuth client secrets, private keys, and webhook secrets belong
  in an environment/secret manager, never in source, frontend bundles, logs, or
  documentation.

## Operational surfaces

- API documentation is disabled by default; enable it only for local
  development or an intentionally protected environment.
- Admin routes, `/sentry-debug`, and metrics/reset operations are operational
  surfaces, not public product APIs.
- Keep CORS origins narrow for deployed environments.
- Run the health check and inspect worker readiness, not just HTTP liveness.
