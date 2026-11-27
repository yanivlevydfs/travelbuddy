# Feature Documentation Index

This index maps the implemented product areas to their detailed guides,
important API surfaces, and relevant engineering documents. Features that call
external services require their configuration; optional route families are
called out below.

| Area | What it covers | Detailed guide |
| --- | --- | --- |
| Trip planning | AI planning, multi-stop journeys, and TripDoc v3 | [Trip planning](./trip_planning.md) |
| Search and recommendations | Travel search, ranking, personalization, and feedback | [Search and recommendations](./search_and_recommendations.md) |
| Reels | Asynchronous video-to-location pipeline | [Reel intelligence](./reels.md) |
| Marketplace and community | Partner offers, checkout, traveller trips, and groups | [Marketplace and community](./marketplace_and_community.md) |
| HostOS / StayOS | Hotel PMS, inventory, reservations, billing, and websites | [HostOS PMS](./hostos_pms.md) |
| Guest messaging and marketing | AI employees, WhatsApp/Instagram, campaigns, and ads | [Messaging and marketing](./messaging_and_marketing.md) |
| Platform APIs | Discovery, onboarding, admin, metrics, and user surfaces | [Platform and admin](./platform_and_admin.md) |

## Runtime modes

- **Travel application (default):** trip planning, search, recommendations,
  Reels, marketplace, groups, referral, and traveller-facing hotel/hostel
  catalog APIs.
- **HostOS / StayOS (optional):** PMS, messaging, and related hotel-operation
  routes. Enable with `ENABLE_HOTEL_OS=true` only when the process points to the
  hotel-operations Supabase project.
- **Google Business Profile (optional):** its routes additionally require
  `ENABLE_GOOGLE_BUSINESS_PROFILE=true` and Google OAuth configuration.

For the concrete route families, see the [API surface reference](../api_surface.md).
For infrastructure and credentials, see [integration.md](../../integration.md).

## Existing deeper design documents

- [Trip engine architecture](../trip_engine_architecture.md)
- [Trip graph](../trip_graph.md)
- [Recommendation engine](../recommendation_engine.md)
- [Persistence architecture](../persistence_architecture.md)
- [Agent architecture map](../agent_architecture_map.md)
- [Agent arbitration](../agent_arbitration.md)
