# TravBuddy Product Overview

## What it is

TravBuddy is a travel technology backend for trip planning, travel search and
recommendations, video-to-location discovery, travel commerce, and hotel
operations. It combines AI-assisted workflows with provider data and
application-owned business rules.

This repository contains backend source code, database schema and migrations,
workers, scripts, tests, and engineering documentation. It does not include the
traveller or hotel-operator frontend.

## Product areas

- **Trip planning:** Plan multi-stop journeys with transport, stays, and
  day-by-day itinerary content. TripDoc stores a generated plan and supports
  changing selections without regenerating the entire trip.
- **Search and recommendations:** Aggregate travel inventory, filter it against
  user constraints, score candidates, and optionally rerank or explain results
  with AI.
- **Video location intelligence:** Turn an Instagram Reel into a timeline of
  likely locations, with asynchronous processing and progress updates.
- **Marketplace and community:** Extract partner offers, match inventory, take
  payments, and support saved trips and group travel.
- **HostOS / StayOS:** An optional hotel operations mode for reservations,
  front-desk workflows, folios, hotel websites, and AI-assisted guest messaging.

See [Features.md](./Features.md) for the catalogue and
[docs/features/index.md](./docs/features/index.md) for feature-by-feature
guides.

## Intended users

- Travellers planning multi-city journeys and comparing transport or stays.
- Travel businesses and partners publishing offers or inventory.
- Hotel teams managing reservations, guest operations, and digital channels.
- Engineers building or studying AI, travel aggregation, recommendation, and
  multi-tenant SaaS systems.

## Product status

The hosted TravBuddy services were retired in September 2026. This is an
open-source codebase intended for self-hosting, learning, and further
development; it is not a currently operated production service. Restoring
features requires configuring the relevant third-party services and credentials.
See [SHUTDOWN.md](./SHUTDOWN.md) for the infrastructure status.

## Operating modes

The default deployment serves the travel planner, search, recommendations,
Reels, marketplace, and community APIs. HostOS / StayOS is optional and is
enabled with `ENABLE_HOTEL_OS=true`; it is designed for a separate Supabase
project and deployment configuration.

## Product boundaries

- Results from external providers depend on provider availability, access
  controls, and their terms.
- AI output is not the source of truth for inventory, prices, payments, or
  booking state; application services and provider responses own those facts.
- A working API process does not imply all integrations are configured. Check
  `/api/v1/health` and the [integration guide](./integration.md).
