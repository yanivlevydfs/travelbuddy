# Marketplace, Partner Offers, and Community Trips

## Partner marketplace

Partner-facing marketplace routes support:

- Extracting structured listings from uploaded files or supplied text.
- Searching for relevant places and matching offers to inventory.
- Quoting prices, uploading media, submitting listings, and publishing or
  unpublishing inventory.
- Recording views and AI-related job usage.
- Creating and looking up bookings.
- Starting, verifying, cancelling, and applying coupons to subscription plans.
- Processing Razorpay webhooks.

The principal route family is `/api/v1/marketplace/*`. Relevant implementation
modules include `app/api/v1/marketplace/` and `app/services/marketplace_*`.
Razorpay credentials and webhook configuration are described in
[integration.md](../../integration.md).

## Traveller trips and bookings

The `/api/v1/trips/{trip_id}/*` APIs manage trip bookings, documents, uploads,
Reels, and saved Reel attachments. These user-scoped traveller APIs are distinct
from partner listing management and from TripDoc v3.

## Groups

Group routes under `/api/v1/groups` provide:

- Group creation, lookup, invite preview, join, and lookup by trip.
- Messages and AI assistance.
- Shared expenses and expense deletion.
- Poll voting on supported group messages.

Group-travel inventory is also exposed through `/api/v1/discover/*` and
`/api/v1/marketplace/trips/*` operations for listing publication, trip seats,
and departure seats.

## Referrals, credits, and discovery

- `/api/v1/referral/me` and `/api/v1/referral/redeem` manage referral status and
  code redemption.
- `/api/v1/community/live` and `/api/v1/community/recent` expose community
  activity.
- `/api/v1/discover/vibe-trips` and `/api/v1/discover/weekend-plans` return
  discovery content.
- Admin routes include coupon, credit-ledger, partner, payout, listing, and trip
  operations.

## Data and dependencies

The core schema and migrations define traveller trip, booking, group, referral,
credit, marketplace, and search-store data. Payment workflows need Razorpay;
media upload workflows need the configured storage backend. Most writes require
Supabase service access and authenticated user or partner context.
