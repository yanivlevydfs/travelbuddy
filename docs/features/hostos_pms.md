# HostOS / StayOS Hotel Operations

HostOS is the optional hotel-operations deployment mode. Its routes are mounted
only when `ENABLE_HOTEL_OS=true`; the process must point to the HostOS Supabase
project, not the traveller/marketplace project. See
[DEPLOYING_STAYOS.md](../../DEPLOYING_STAYOS.md).

## Property and inventory setup

The PMS APIs support property onboarding, room types, rooms, rate plans,
readiness checks, inventory, availability, rates, and synchronization. Property
configuration includes services, meals, experiences, group tiers, stay-time
rules, negotiation rules, and notification policies.

Main route family: `/api/v1/pms/properties/{property_id}/*`.

## Reservations and front desk

The PMS includes:

- Quote creation and reservation lookup/creation.
- Guest check-in and check-out.
- Folio reads and adjustments.
- Payment-link creation and rotation.
- Room status, tasks, night audit, and registration data.
- Public booking, stay, quote, and payment endpoints for hotel websites.

The pricing, availability, reservation, and payment logic is implemented in
`app/services/pms/`. Provider/payment callbacks are not a substitute for
application-owned booking state.

## Staff, access, and operator tools

- Property membership and role-based permissions are resolved server-side.
- Owner, manager, accounts, receptionist, housekeeping, maintenance,
  concierge, restaurant, and driver roles have distinct permissions.
- Owner, front-desk, and marketing AI-assisted tools are exposed through
  property-scoped routes.
- Agent actions, approval decisions, usage, billing, analytics, recommendations,
  and escalations are available to appropriately authorized members.

See [Security and tenancy](../security_and_tenancy.md) for identity and
property-isolation behavior.

## Hotel websites and distribution

Hotel website routes support editing, inventory projection, generation,
rewriting, publishing, domain setup/verification, and public host lookup.
Public hotel routes support discovery, availability, quoting, ordering, and
payment confirmation. These operations depend on a working PMS database and,
for paid workflows, Razorpay configuration.

## Agency and creator workflows

- Agency routes support contracted availability, quotes, bookings, and booking
  lookup.
- Creator campaign routes support campaign setup, approval/status, touch and
  payout views.
- Marketing profile and ad routes manage audiences, drafts, creative, campaigns,
  and publishing workflows.

## Deployment constraints

- HostOS uses a different Supabase database from the default travel deployment.
- The `hotelos` Celery queue is reserved for hotel messaging, marketing, and PMS
  tasks.
- Do not mount HostOS routes against the travel project's database.
- Channel credentials require `MESSAGING_CREDENTIALS_KEY`; configure it before
  connecting WhatsApp or Instagram.
