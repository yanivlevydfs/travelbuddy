# Database Migration Index

The SQL files in [`migrations/`](../migrations/) are the executable migration
history. This index groups them by feature; inspect each SQL file before
applying it. Some migrations assume that tables already exist in the target
Supabase project.

## Base schema

- [`schema.sql`](../schema.sql) — core traveller schema, helper functions, the
  auth profile trigger, and row-level security policies.

## Trip planning, search, and recommendations

- `2026-07-13_reco_engine.sql` — `pgvector`, hotel profile/embedding and
  traveller profile tables, transport observations, feedback, ranking
  aggregates, and the `match_hotels` RPC.
- `2026-07-14_search_store.sql` — immutable search batches/results, mutable
  result views, precomputed recommendations, and per-trip memory. This design
  was later removed by the TripDoc teardown migration below.
- `2026-07-16_trip_docs.sql` — TripDoc-related schema changes.
- `2026-07-16_trip_snapshots.sql` — trip snapshot storage.
- `2026-07-16_drop_tripstate_tables.sql` — **destructive cleanup**, recorded as
  applied on 2026-07-16. It drops the old event log, copilot transcript,
  search-store/view/recommendation/memory tables and legacy TripState columns.
  It leaves `trip_states.doc` and `trip_snapshots` as the TripDoc persistence
  path. Inspect its exact statements and target database before applying it to
  any other project.

## Groups, referrals, and marketplace

- `2026-07-16_group_trips.sql` — group-trip inventory.
- `2026-07-16_referral_credits.sql` — referral codes and credit ledger.
- `2026-08-04_partner_deals.sql` — partner commercial terms and booking
  commission snapshots.
- `2026-08-04_partner_deals_hardening.sql` — hardening for partner deals.
- `2026-08-22_razorpay_webhook_events_rls.sql` — webhook-event access policies.

## HostOS / StayOS

These dated migrations add or refine hotel operating rules, staff, messaging
escalation, marketing, and guest-facing inventory:

- `2026-08-23_a_quote_remembers_which_rule_priced_it.sql`
- `2026-08-23_a_room_has_a_floor_not_just_a_ceiling.sql`
- `2026-08-23_a_selling_agent_can_answer_the_questions_guests_ask.sql`
- `2026-08-23_an_agent_rate_can_be_a_discount_from_the_public_one.sql`
- `2026-08-23_an_escalation_is_a_thing_that_happened.sql`
- `2026-08-23_breakfast_is_a_fact_not_a_sentence.sql`
- `2026-08-23_hotel_decides_what_a_late_checkout_costs.sql`
- `2026-08-23_marketing_that_can_be_reused.sql`
- `2026-08-23_room_described_well_enough_to_sell.sql`
- `2026-08-23_the_hotel_team_is_people_not_logins.sql`
- `2026-08-23_the_platform_fee_is_a_number_not_a_conversation.sql`
- `2026-08-23_three_rooms_is_a_different_price_from_one.sql`
- `2026-08-23_what_a_hotel_sells_besides_a_bed.sql`
- `2026-08-23_who_changed_the_floor_price.sql`
- `2026-08-24_a_lookup_is_not_a_negotiation.sql`
- `2026-08-24_a_model_nobody_priced_is_free.sql`
- `2026-08-24_a_required_action_is_not_optional.sql`
- `2026-08-24_answering_a_message_twice_is_not_two_bookings.sql`
- `2026-08-26_a_template_is_a_look_not_a_property_type.sql`

See [Database schema reference](./database_schema.md) for the table domains
these changes affect and [Deploying StayOS](../DEPLOYING_STAYOS.md) for
deployment-specific database separation.

## Applying and verifying

The README's local setup describes applying `schema.sql` and the migrations
needed for the selected features. The repository does not make every migration
automatically safe for every Supabase project. Before applying:

- Confirm whether the target is the travel or HostOS project.
- Check whether the migration's prerequisite tables and columns exist.
- Review every `DROP`, `DELETE`, `UPDATE`, backfill, and policy replacement.
- Back up data before destructive changes.
- Apply to a disposable or staging project first.
- Run the relevant tests and feature-specific verification scripts.

Never apply a migration to a hosted project merely because it appears in this
index.
