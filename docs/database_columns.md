# Database Column Catalog

This catalog lists column declarations from every `CREATE TABLE` statement
checked into `schema.sql` and `migrations/*.sql` (55 table definitions at
generation time). Exact DDL, indexes, policies, and later `ALTER TABLE`
changes remain authoritative in those SQL files. Refresh this page when
the SQL schema changes.

> **Current-state warning:** some definitions below are historical. The
> TripDoc migration on 2026-07-16 dropped the legacy event/copilot/search
> tables listed in the status notes and changed `trip_states`. HostOS base
> tables such as `properties` and `reservations` are referenced by the app
> but are not created by the DDL set covered here.

## `profiles`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE` |
| `full_name` | `text` |
| `username` | `text` |
| `email` | `text` |
| `bio` | `text` |
| `avatar_url` | `text` |
| `city` | `text` |
| `country` | `text` |
| `gender` | `text` |
| `age` | `integer` |
| `phone_number` | `text` |
| `language` | `text DEFAULT 'en'` |
| `currency` | `text DEFAULT 'inr'` |
| `role` | `text DEFAULT 'user'` |
| `vibes` | `text[] DEFAULT '{}'` |
| `preferences` | `jsonb DEFAULT '{}'` |
| `social_links` | `jsonb DEFAULT '{}'` |
| `has_early_access` | `boolean DEFAULT false` |
| `onboarding_completed` | `boolean DEFAULT false` |
| `updated_at` | `timestamptz` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `trips`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `title` | `text NOT NULL` |
| `destination` | `text` |
| `start_date` | `date` |
| `end_date` | `date` |
| `budget` | `numeric` |
| `currency` | `text DEFAULT 'INR'` |
| `description` | `text` |
| `image_url` | `text` |
| `itinerary_data` | `jsonb DEFAULT '{}'` |
| `is_public` | `boolean DEFAULT false` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `internal_trip_logs`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid REFERENCES auth.users(id) ON DELETE CASCADE` |
| `destination` | `text` |
| `days` | `integer` |
| `budget` | `text` |
| `interests` | `jsonb` |
| `itinerary_data` | `jsonb` |
| `is_public` | `boolean DEFAULT true` |
| `created_at` | `timestamptz DEFAULT now()` |

## `trip_groups`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `text` |
| `name` | `text NOT NULL` |
| `destination` | `text` |
| `created_by` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `invite_code` | `text NOT NULL` |
| `created_at` | `timestamptz DEFAULT now()` |

## `group_members`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `group_id` | `uuid REFERENCES public.trip_groups(id) ON DELETE CASCADE` |
| `user_id` | `uuid REFERENCES auth.users(id) ON DELETE CASCADE` |
| `display_name` | `text` |
| `role` | `text DEFAULT 'member' CHECK (role IN ('admin', 'member'))` |
| `joined_at` | `timestamptz DEFAULT now()` |

## `group_messages`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `group_id` | `uuid NOT NULL REFERENCES public.trip_groups(id) ON DELETE CASCADE` |
| `user_id` | `uuid REFERENCES auth.users(id) ON DELETE SET NULL` |
| `sender_name` | `text NOT NULL` |
| `text` | `text NOT NULL` |
| `type` | `text NOT NULL DEFAULT 'text'` |
| `poll_data` | `jsonb` |
| `created_at` | `timestamptz DEFAULT now()` |

## `group_expenses`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `group_id` | `uuid NOT NULL REFERENCES public.trip_groups(id) ON DELETE CASCADE` |
| `title` | `text NOT NULL` |
| `amount` | `numeric NOT NULL` |
| `paid_by` | `text NOT NULL` |
| `paid_by_user_id` | `uuid REFERENCES auth.users(id) ON DELETE SET NULL` |
| `split_between` | `text[] DEFAULT '{}'` |
| `category` | `text DEFAULT 'Other'` |
| `date` | `text` |
| `created_at` | `timestamptz DEFAULT now()` |

## `trip_bookings`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `text NOT NULL` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `type` | `text NOT NULL` |
| `category` | `text` |
| `title` | `text NOT NULL` |
| `description` | `text` |
| `details` | `text` |
| `status` | `text DEFAULT 'Planned'` |
| `price` | `text` |
| `price_unit` | `text` |
| `file_gcs_path` | `text` |
| `file_gcs_url` | `text` |
| `has_file` | `boolean DEFAULT false` |
| `attachment_name` | `text` |
| `created_at` | `timestamptz DEFAULT now()` |

## `trip_documents`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `text NOT NULL` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `name` | `text NOT NULL` |
| `category` | `text NOT NULL` |
| `file_name` | `text NOT NULL` |
| `gcs_path` | `text NOT NULL` |
| `gcs_url` | `text NOT NULL` |
| `status` | `text DEFAULT 'pending'` |
| `created_at` | `timestamptz DEFAULT now()` |

## `trip_reels`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `text NOT NULL` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `title` | `text NOT NULL` |
| `category` | `text DEFAULT 'Memory'` |
| `gcs_path` | `text NOT NULL` |
| `gcs_url` | `text NOT NULL` |
| `thumbnail_url` | `text` |
| `duration` | `text` |
| `status` | `text DEFAULT 'pending'` |
| `created_at` | `timestamptz DEFAULT now()` |

## `reel_scans`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid REFERENCES auth.users(id) ON DELETE CASCADE` |
| `job_id` | `text UNIQUE NOT NULL` |
| `reel_url` | `text NOT NULL` |
| `caption` | `text` |
| `video_url` | `text` |
| `frame_urls` | `text[] DEFAULT '{}'` |
| `result` | `jsonb` |
| `status` | `text NOT NULL DEFAULT 'queued'` |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` |
| `completed_at` | `timestamptz` |

## `reel_jobs`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `job_id` | `text` |
| `source_url` | `text` |
| `source_title` | `text` |
| `title` | `text` |
| `thumbnail_url` | `text` |
| `video_url` | `text` |
| `status` | `text NOT NULL DEFAULT 'pending'` |
| `progress` | `integer DEFAULT 0` |
| `error_message` | `text` |
| `created_at` | `timestamptz DEFAULT now()` |
| `updated_at` | `timestamptz DEFAULT now()` |

Table-level constraints:
- `UNIQUE (user_id, job_id)`

## `saved_reels`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `job_id` | `text` |
| `title` | `text NOT NULL` |
| `category` | `text NOT NULL` |
| `thumbnail_url` | `text DEFAULT ''` |
| `video_url` | `text DEFAULT ''` |
| `description` | `text DEFAULT ''` |
| `vibes` | `text[] DEFAULT '{}'` |
| `is_liked` | `boolean DEFAULT false` |
| `created_at` | `timestamptz DEFAULT now()` |

## `trip_saved_reels`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `trip_id` | `text NOT NULL` |
| `reel_id` | `uuid NOT NULL REFERENCES public.saved_reels(id) ON DELETE CASCADE` |
| `created_at` | `timestamptz DEFAULT now()` |

Table-level constraints:
- `UNIQUE (user_id, trip_id, reel_id)`

## `trip_journey`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `uuid NOT NULL REFERENCES public.trips(id) ON DELETE CASCADE` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `origin` | `text` |
| `destination` | `text` |
| `departure_date` | `date` |
| `return_date` | `date` |
| `num_days` | `integer` |
| `travelers` | `integer` |
| `budget` | `text` |
| `stay_nights` | `integer` |
| `going` | `jsonb DEFAULT '{}'` |
| `return_journey` | `jsonb DEFAULT '{}'` |
| `journey_summary` | `text` |
| `total_distance_km` | `numeric` |
| `total_duration_approx` | `text` |
| `total_estimated_cost` | `numeric` |
| `cost_per_person` | `numeric` |
| `cost_for_all_travelers` | `numeric` |
| `cost_breakdown` | `jsonb DEFAULT '{}'` |
| `budget_breakdown` | `jsonb DEFAULT '{}'` |
| `best_time_to_depart` | `text` |
| `packing_tips` | `jsonb DEFAULT '[]'` |
| `is_international` | `boolean DEFAULT false` |
| `visa_required` | `boolean DEFAULT false` |
| `visa_info` | `text` |
| `documents_needed` | `jsonb DEFAULT '[]'` |
| `health_requirements` | `jsonb DEFAULT '[]'` |
| `currency_info` | `jsonb DEFAULT '{}'` |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` |

## `saved_folders`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `name` | `text NOT NULL` |
| `emoji` | `text NOT NULL DEFAULT '📁'` |
| `item_type` | `text NOT NULL DEFAULT 'mixed' CHECK (item_type IN ('trips','nights','reels','mixed'))` |
| `created_at` | `timestamptz NOT NULL DEFAULT now()` |
| `updated_at` | `timestamptz NOT NULL DEFAULT now()` |

## `folder_items`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `folder_id` | `uuid NOT NULL REFERENCES public.saved_folders(id) ON DELETE CASCADE` |
| `item_id` | `text NOT NULL` |
| `item_type` | `text NOT NULL CHECK (item_type IN ('trip','night','reel'))` |
| `item_snapshot` | `jsonb NOT NULL DEFAULT '{}'` |
| `added_at` | `timestamptz NOT NULL DEFAULT now()` |

Table-level constraints:
- `UNIQUE (folder_id, item_id, item_type)`

## `user_home_feed`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `feed_type` | `text NOT NULL` |
| `items` | `jsonb NOT NULL DEFAULT '[]'` |
| `generated_at` | `timestamptz DEFAULT now()` |

## `trip_states`

Source: [`schema.sql`](../schema.sql). Base definition shown; current TripDoc adds `doc` and `progress`, then drops `state`, `last_seq`, and `schema_version`.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid REFERENCES auth.users(id) ON DELETE CASCADE` |
| `title` | `text` |
| `origin` | `text` |
| `destination` | `text` |
| `status` | `text NOT NULL DEFAULT 'planning'` |
| `visibility` | `text NOT NULL DEFAULT 'private'` |
| `schema_version` | `integer NOT NULL DEFAULT 1` |
| `state` | `jsonb NOT NULL DEFAULT '{}'` |
| `last_seq` | `integer NOT NULL DEFAULT 0` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |
| `updated_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `trip_state_events`

Source: [`schema.sql`](../schema.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `id` | `bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `seq` | `integer NOT NULL` |
| `actor` | `text NOT NULL` |
| `phase` | `text` |
| `patches` | `jsonb NOT NULL DEFAULT '[]'` |
| `reasoning` | `text` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

Table-level constraints:
- `UNIQUE (trip_id, seq)`

## `copilot_messages`

Source: [`schema.sql`](../schema.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `id` | `bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `role` | `text NOT NULL` |
| `content` | `text NOT NULL` |
| `intent` | `text` |
| `meta` | `jsonb NOT NULL DEFAULT '{}'` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `user_travel_memory`

Source: [`schema.sql`](../schema.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `user_id` | `uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE` |
| `memory` | `jsonb NOT NULL DEFAULT '{}'` |
| `updated_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `hotel_profiles`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `hotel_id` | `uuid PRIMARY KEY REFERENCES public.hotels(id) ON DELETE CASCADE` |
| `personality_tags` | `text[] NOT NULL DEFAULT '{}'` |
| `one_liner` | `text` |
| `seo_summary` | `text` |
| `highlights` | `text[] DEFAULT '{}'` |
| `best_for` | `text[] DEFAULT '{}'` |
| `not_for` | `text[] DEFAULT '{}'` |
| `confidence` | `real` |
| `source_review_count` | `int` |
| `generated_by` | `text` |
| `refreshed_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `hotel_embeddings`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `hotel_id` | `uuid PRIMARY KEY REFERENCES public.hotels(id) ON DELETE CASCADE` |
| `embedding` | `vector(384) NOT NULL` |
| `doc_hash` | `text` |
| `model` | `text` |
| `updated_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `personality_centroids`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `tag` | `text PRIMARY KEY` |
| `centroid` | `vector(384) NOT NULL` |
| `n_exemplars` | `int` |
| `updated_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `traveller_profiles`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `user_id` | `uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE` |
| `axes` | `jsonb` |
| `embedding` | `vector(384)` |
| `updated_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `transport_options`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `mode` | `text NOT NULL` |
| `source` | `text` |
| `origin` | `text` |
| `destination` | `text` |
| `depart_date` | `date` |
| `operator` | `text` |
| `operator_code` | `text` |
| `service_no` | `text` |
| `depart_time` | `text` |
| `arrive_time` | `text` |
| `duration_min` | `int` |
| `stops` | `int` |
| `price` | `numeric` |
| `currency` | `text DEFAULT 'INR'` |
| `fare_type` | `text` |
| `baggage_kg` | `int` |
| `cabin_baggage_kg` | `int` |
| `layover_airports` | `text[]` |
| `seats_available` | `int` |
| `refundable` | `boolean` |
| `raw` | `jsonb` |
| `scraped_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `rec_feedback`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid` |
| `trip_id` | `uuid` |
| `session_id` | `text` |
| `item_type` | `text` |
| `item_id` | `text` |
| `event` | `text NOT NULL` |
| `position` | `int` |
| `dwell_ms` | `int` |
| `context` | `jsonb` |
| `created_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `ranking_weights`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `persona` | `text` |
| `feature` | `text` |
| `weight` | `real` |
| `variant` | `text DEFAULT 'control'` |
| `updated_at` | `timestamptz DEFAULT timezone('utc', now())` |

Table-level constraints:
- `PRIMARY KEY (persona, feature, variant)`

## `expected_price`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `scope` | `text` |
| `key` | `text` |
| `p10` | `numeric` |
| `p50` | `numeric` |
| `p90` | `numeric` |
| `n` | `int` |
| `computed_at` | `timestamptz DEFAULT timezone('utc', now())` |

Table-level constraints:
- `PRIMARY KEY (scope, key)`

## `destination_priors`

Source: [`2026-07-13_reco_engine.sql`](../migrations/2026-07-13_reco_engine.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `destination` | `text PRIMARY KEY` |
| `top_personas` | `text[]` |
| `top_vibes` | `text[]` |
| `seed_embedding` | `vector(384)` |
| `updated_at` | `timestamptz DEFAULT timezone('utc', now())` |

## `search_batches`

Source: [`2026-07-14_search_store.sql`](../migrations/2026-07-14_search_store.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `kind` | `text NOT NULL` |
| `scope_id` | `text NOT NULL DEFAULT ''` |
| `mode` | `text` |
| `params_hash` | `text NOT NULL` |
| `origin` | `text` |
| `destination` | `text` |
| `depart_date` | `date` |
| `status` | `text NOT NULL DEFAULT 'ready'` |
| `result_count` | `int NOT NULL DEFAULT 0` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |
| `expires_at` | `timestamptz` |

## `search_results`

Source: [`2026-07-14_search_store.sql`](../migrations/2026-07-14_search_store.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `batch_id` | `uuid NOT NULL REFERENCES public.search_batches(id) ON DELETE CASCADE` |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `item_type` | `text NOT NULL` |
| `ext_id` | `text` |
| `operator` | `text` |
| `price` | `numeric` |
| `currency` | `text DEFAULT 'INR'` |
| `departs_at` | `timestamptz` |
| `arrives_at` | `timestamptz` |
| `duration_min` | `int` |
| `stops` | `int` |
| `rating` | `numeric` |
| `refundable` | `boolean` |
| `summary` | `jsonb NOT NULL DEFAULT '{}'` |
| `raw` | `jsonb` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `trip_result_views`

Source: [`2026-07-14_search_store.sql`](../migrations/2026-07-14_search_store.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `result_id` | `uuid NOT NULL REFERENCES public.search_results(id) ON DELETE CASCADE` |
| `visible` | `boolean` |
| `favorite` | `boolean NOT NULL DEFAULT false` |
| `selected` | `boolean NOT NULL DEFAULT false` |
| `score_override` | `real` |
| `notes` | `text` |
| `source` | `text NOT NULL DEFAULT 'user'` |
| `updated_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

Table-level constraints:
- `PRIMARY KEY (trip_id, result_id)`

## `trip_recommendations`

Source: [`2026-07-14_search_store.sql`](../migrations/2026-07-14_search_store.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `scope_id` | `text NOT NULL DEFAULT ''` |
| `batch_id` | `uuid REFERENCES public.search_batches(id) ON DELETE CASCADE` |
| `result_id` | `uuid NOT NULL REFERENCES public.search_results(id) ON DELETE CASCADE` |
| `score` | `real NOT NULL` |
| `subscores` | `jsonb NOT NULL DEFAULT '{}'` |
| `explanation` | `text` |
| `signals` | `jsonb NOT NULL DEFAULT '{}'` |
| `persona` | `text NOT NULL DEFAULT 'default'` |
| `computed_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

Table-level constraints:
- `PRIMARY KEY (trip_id, result_id, persona)`

## `trip_memory`

Source: [`2026-07-14_search_store.sql`](../migrations/2026-07-14_search_store.sql). Historical ? dropped by `2026-07-16_drop_tripstate_tables.sql`.

| Column | SQL declaration |
| --- | --- |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `key` | `text NOT NULL` |
| `scope` | `text NOT NULL DEFAULT ''` |
| `value` | `jsonb NOT NULL` |
| `confidence` | `real NOT NULL DEFAULT 0.6` |
| `source` | `text NOT NULL DEFAULT 'copilot'` |
| `updated_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

Table-level constraints:
- `PRIMARY KEY (trip_id, key, scope)`

## `group_trips`

Source: [`2026-07-16_group_trips.sql`](../migrations/2026-07-16_group_trips.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `slug` | `text NOT NULL UNIQUE` |
| `title` | `text NOT NULL` |
| `seo_title` | `text` |
| `description` | `text` |
| `destination` | `text` |
| `origin` | `text` |
| `duration_label` | `text` |
| `duration_days` | `int` |
| `departure_date` | `date` |
| `departure_label` | `text` |
| `seats_total` | `int` |
| `seats_left` | `int` |
| `price_from` | `int` |
| `whatsapp_number` | `text` |
| `hero_image` | `text` |
| `content` | `jsonb NOT NULL DEFAULT '{}'::jsonb` |
| `is_active` | `boolean NOT NULL DEFAULT true` |
| `sort` | `int NOT NULL DEFAULT 0` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |
| `updated_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `referral_codes`

Source: [`2026-07-16_referral_credits.sql`](../migrations/2026-07-16_referral_credits.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `user_id` | `uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE` |
| `code` | `text NOT NULL UNIQUE` |
| `member_no` | `bigint GENERATED ALWAYS AS IDENTITY` |
| `display_name` | `text` |
| `referred_by` | `uuid REFERENCES auth.users(id)` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `credit_ledger`

Source: [`2026-07-16_referral_credits.sql`](../migrations/2026-07-16_referral_credits.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `user_id` | `uuid NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE` |
| `amount` | `int NOT NULL` |
| `reason` | `text NOT NULL` |
| `ref_id` | `text` |
| `note` | `text` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `trip_snapshots`

Source: [`2026-07-16_trip_snapshots.sql`](../migrations/2026-07-16_trip_snapshots.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid PRIMARY KEY DEFAULT gen_random_uuid()` |
| `trip_id` | `uuid NOT NULL REFERENCES public.trip_states(id) ON DELETE CASCADE` |
| `generation_id` | `uuid` |
| `reason` | `text NOT NULL DEFAULT ''` |
| `spec_version` | `int NOT NULL DEFAULT 1` |
| `last_seq` | `int` |
| `state` | `jsonb NOT NULL` |
| `created_at` | `timestamptz NOT NULL DEFAULT timezone('utc', now())` |

## `partner_terms`

Source: [`2026-08-04_partner_deals.sql`](../migrations/2026-08-04_partner_deals.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `partner_id` | `uuid primary key references partners(id) on delete cascade` |
| `subscription_fee_paise` | `integer not null default 49900 check (subscription_fee_paise >= 0)` |
| `billing_period` | `text not null default 'yearly' check (billing_period in ('monthly', 'yearly'))` |
| `commission_pct` | `numeric not null default 10 check (commission_pct >= 0 and commission_pct <= 50)` |
| `commission_expires_at` | `timestamptz` |
| `base_commission_pct` | `numeric not null default 10 check (base_commission_pct >= 0 and base_commission_pct <= 50)` |
| `source` | `text not null default 'default' check (source in ('default', 'coupon', 'admin'))` |
| `source_coupon_id` | `uuid references platform_coupons(id) on delete set null` |
| `notes` | `text` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint partner_terms_intro_check check (commission_expires_at is null or commission_pct <> base_commission_pct)`

## `negotiation_rules`

Source: [`2026-08-23_a_room_has_a_floor_not_just_a_ceiling.sql`](../migrations/2026-08-23_a_room_has_a_floor_not_just_a_ceiling.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `scope` | `text not null check (scope in ('property','room_type','rate_plan'))` |
| `room_type_id` | `uuid references public.room_types(id) on delete cascade` |
| `rate_plan_id` | `uuid references public.rate_plans(id) on delete cascade` |
| `enabled` | `boolean not null default true` |
| `floor_pct` | `integer check (floor_pct is null or floor_pct between 0 and 100)` |
| `floor_paise` | `integer check (floor_paise is null or floor_paise >= 0)` |
| `target_pct` | `integer check (target_pct is null or target_pct between 0 and 100)` |
| `target_paise` | `integer check (target_paise is null or target_paise >= 0)` |
| `ceiling_paise` | `integer check (ceiling_paise is null or ceiling_paise >= 0)` |
| `auto_pct` | `integer check (auto_pct is null or auto_pct between 0 and 100)` |
| `approval_pct` | `integer check (approval_pct is null or approval_pct between 0 and 100)` |
| `valid_from` | `date` |
| `valid_to` | `date` |
| `days_of_week` | `smallint[] not null default '{}'` |
| `blackout_dates` | `date[] not null default '{}'` |
| `min_nights` | `integer not null default 1 check (min_nights >= 1)` |
| `min_units` | `integer not null default 1 check (min_units >= 1)` |
| `note` | `text` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint negotiation_rules_scope_target check ( (scope = 'property' and room_type_id is null and rate_plan_id is null) or (scope = 'room_type' and room_type_id is not null and rate_plan_id is null) or (scope = 'rate_plan' and rate_plan_id is not null and room_type_id is null) )`
- `constraint negotiation_rules_ladder check ( auto_pct is null or approval_pct is null or approval_pct >= auto_pct )`

## `meal_options`

Source: [`2026-08-23_breakfast_is_a_fact_not_a_sentence.sql`](../migrations/2026-08-23_breakfast_is_a_fact_not_a_sentence.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `meal` | `text not null check (meal in ('breakfast','lunch','dinner'))` |
| `availability` | `text not null default 'available' check (availability in ('available','on_request','not_available'))` |
| `price_paise` | `integer check (price_paise is null or price_paise >= 0)` |
| `price_unit` | `text not null default 'per_person' check (price_unit in ('per_person','per_room','fixed'))` |
| `starts_at` | `time` |
| `ends_at` | `time` |
| `diet` | `text check (diet is null or diet in ('veg','nonveg','both','jain','vegan'))` |
| `service_style` | `text check (service_style is null or service_style in ('buffet','set_menu','a_la_carte','room_service'))` |
| `cuisine` | `text` |
| `note` | `text` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `unique (property_id, meal)`

## `stay_time_rules`

Source: [`2026-08-23_hotel_decides_what_a_late_checkout_costs.sql`](../migrations/2026-08-23_hotel_decides_what_a_late_checkout_costs.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `kind` | `text not null check (kind in ('early_in','late_out'))` |
| `until_time` | `time not null` |
| `charge_kind` | `text not null check (charge_kind in ('free','fixed','pct_of_night','unavailable'))` |
| `amount` | `integer not null default 0 check (amount >= 0)` |
| `subject_to_availability` | `boolean not null default true` |
| `note` | `text` |
| `is_active` | `boolean not null default true` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `unique (property_id, kind, until_time)`

## `marketing_profiles`

Source: [`2026-08-23_marketing_that_can_be_reused.sql`](../migrations/2026-08-23_marketing_that_can_be_reused.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `property_id` | `uuid primary key references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `usp` | `text` |
| `best_for` | `text[] not null default '{}'` |
| `ideal_guest` | `text` |
| `popular_with` | `text[] not null default '{}'` |
| `nearby` | `jsonb not null default '[]'::jsonb` |
| `best_months` | `smallint[] not null default '{}'` |
| `weekend_demand` | `text check (weekend_demand is null or weekend_demand in ('low','medium','high'))` |
| `weekday_demand` | `text check (weekday_demand is null or weekday_demand in ('low','medium','high'))` |
| `seasonality` | `text` |
| `tone` | `text` |
| `languages` | `text[] not null default '{}'` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

## `ad_audiences`

Source: [`2026-08-23_marketing_that_can_be_reused.sql`](../migrations/2026-08-23_marketing_that_can_be_reused.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `name` | `text not null` |
| `age_min` | `smallint check (age_min is null or age_min between 13 and 65)` |
| `age_max` | `smallint check (age_max is null or age_max between 13 and 65)` |
| `genders` | `text[] not null default '{}'` |
| `locations` | `jsonb not null default '[]'::jsonb` |
| `interests` | `text[] not null default '{}'` |
| `travel_intent` | `text[] not null default '{}'` |
| `meta_spec` | `jsonb` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `unique (property_id, name)`
- `constraint ad_audiences_age_range check (age_min is null or age_max is null or age_max >= age_min)`

## `ad_campaigns`

Source: [`2026-08-23_marketing_that_can_be_reused.sql`](../migrations/2026-08-23_marketing_that_can_be_reused.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `name` | `text not null` |
| `channel` | `text not null check (channel in ('instagram','facebook','whatsapp'))` |
| `objective` | `text not null check (objective in ( 'messages','traffic','awareness','engagement','leads'))` |
| `destination` | `text not null default 'whatsapp' check (destination in ('whatsapp','instagram_dm','website','hotel_page'))` |
| `audience_id` | `uuid references public.ad_audiences(id) on delete set null` |
| `room_type_id` | `uuid references public.room_types(id) on delete set null` |
| `experience_id` | `uuid references public.experiences(id) on delete set null` |
| `offer` | `jsonb not null default '{}'::jsonb` |
| `starts_on` | `date` |
| `ends_on` | `date` |
| `daily_budget_paise` | `integer check (daily_budget_paise is null or daily_budget_paise >= 0)` |
| `status` | `text not null default 'draft' check (status in ( 'draft','connected','published','running','paused','failed'))` |
| `external_id` | `text` |
| `external_ref` | `jsonb not null default '{}'::jsonb` |
| `last_error` | `text` |
| `last_synced_at` | `timestamptz` |
| `created_by_member` | `uuid` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint ad_campaigns_dates check (ends_on is null or starts_on is null or ends_on >= starts_on)`
- `constraint ad_campaigns_published_has_id check ( status in ('draft','connected','failed') or external_id is not null )`

## `ad_creatives`

Source: [`2026-08-23_marketing_that_can_be_reused.sql`](../migrations/2026-08-23_marketing_that_can_be_reused.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `campaign_id` | `uuid not null references public.ad_campaigns(id) on delete cascade` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `format` | `text not null check (format in ('single_image','carousel','video','existing_post'))` |
| `headline` | `text` |
| `primary_text` | `text` |
| `cta` | `text` |
| `media` | `jsonb not null default '[]'::jsonb` |
| `instagram_media_id` | `text` |
| `template_key` | `text` |
| `edited` | `boolean not null default false` |
| `is_primary` | `boolean not null default true` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

## `hotel_team_members`

Source: [`2026-08-23_the_hotel_team_is_people_not_logins.sql`](../migrations/2026-08-23_the_hotel_team_is_people_not_logins.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `member_id` | `uuid references public.partner_members(id) on delete set null` |
| `name` | `text not null` |
| `role` | `text not null check (role in ( 'owner','manager','front_desk','reservations', 'sales','accounts','marketing','operations','other'))` |
| `department` | `text` |
| `email` | `text` |
| `whatsapp` | `text` |
| `phone` | `text` |
| `is_active` | `boolean not null default true` |
| `sort_order` | `integer not null default 0` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint hotel_team_members_reachable check ( whatsapp is not null or email is not null or phone is not null or member_id is not null )`

## `team_notification_rules`

Source: [`2026-08-23_the_hotel_team_is_people_not_logins.sql`](../migrations/2026-08-23_the_hotel_team_is_people_not_logins.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `event` | `text not null check (event in ( 'booking_created','payment_received','payment_failed', 'group_inquiry','negotiation_approval','special_request', 'transfer_request','cancellation','refund', 'high_value_booking','vip_guest','unanswered_conversation', 'agent_escalation'))` |
| `team_member_id` | `uuid not null references public.hotel_team_members(id) on delete cascade` |
| `channels` | `text[] not null default '{whatsapp}'` |
| `is_active` | `boolean not null default true` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `unique (property_id, event, team_member_id)`
- `constraint team_notification_rules_channels check ( array_length(channels, 1) >= 1 and channels <@ array['whatsapp','email','dashboard']::text[] )`

## `agent_escalations`

Source: [`2026-08-23_the_hotel_team_is_people_not_logins.sql`](../migrations/2026-08-23_the_hotel_team_is_people_not_logins.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `conversation_id` | `uuid references public.conversations(id) on delete cascade` |
| `reservation_id` | `uuid references public.reservations(id) on delete set null` |
| `quote_id` | `uuid references public.quotes(id) on delete set null` |
| `agent_action_id` | `uuid references public.agent_actions(id) on delete set null` |
| `task_id` | `uuid references public.tasks(id) on delete set null` |
| `kind` | `text not null check (kind in ( 'discount_request','group_inquiry','special_request', 'transfer_request','payment_issue','cancellation_exception', 'unanswered','other'))` |
| `dedupe_key` | `text not null default ''` |
| `reason` | `text` |
| `payload` | `jsonb not null default '{}'::jsonb` |
| `team_member_id` | `uuid references public.hotel_team_members(id) on delete set null` |
| `message_sent` | `text` |
| `status` | `text not null default 'open' check (status in ('open','notified','responded','resolved','dismissed'))` |
| `resolution` | `text` |
| `notified_at` | `timestamptz` |
| `responded_at` | `timestamptz` |
| `resolved_at` | `timestamptz` |
| `resolved_by` | `uuid` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

## `group_rate_tiers`

Source: [`2026-08-23_three_rooms_is_a_different_price_from_one.sql`](../migrations/2026-08-23_three_rooms_is_a_different_price_from_one.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `room_type_id` | `uuid references public.room_types(id) on delete cascade` |
| `min_units` | `integer not null check (min_units >= 1)` |
| `max_units` | `integer check (max_units is null or max_units >= 1)` |
| `discount_pct` | `integer check (discount_pct is null or discount_pct between 0 and 100)` |
| `fixed_price_paise` | `integer check (fixed_price_paise is null or fixed_price_paise >= 0)` |
| `negotiable` | `boolean not null default false` |
| `is_active` | `boolean not null default true` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint group_rate_tiers_range check (max_units is null or max_units >= min_units)`
- `constraint group_rate_tiers_says_something check ( discount_pct is not null or fixed_price_paise is not null or negotiable )`

## `group_policies`

Source: [`2026-08-23_three_rooms_is_a_different_price_from_one.sql`](../migrations/2026-08-23_three_rooms_is_a_different_price_from_one.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `property_id` | `uuid primary key references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `min_units` | `integer not null default 3 check (min_units >= 1)` |
| `deposit_pct` | `integer check (deposit_pct is null or deposit_pct between 0 and 100)` |
| `payment_due_days` | `integer check (payment_due_days is null or payment_due_days >= 0)` |
| `cancellation` | `jsonb not null default '{}'::jsonb` |
| `meal_package` | `text` |
| `blackout_dates` | `date[] not null default '{}'` |
| `terms` | `text` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

## `experiences`

Source: [`2026-08-23_what_a_hotel_sells_besides_a_bed.sql`](../migrations/2026-08-23_what_a_hotel_sells_besides_a_bed.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `uuid primary key default gen_random_uuid()` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `title` | `text not null` |
| `description` | `text` |
| `category` | `text not null default 'other'` |
| `duration_minutes` | `integer check (duration_minutes is null or duration_minutes > 0)` |
| `price_paise` | `integer not null default 0 check (price_paise >= 0)` |
| `price_unit` | `text not null default 'per_person' check (price_unit in ('per_person','per_group','per_vehicle','fixed'))` |
| `meeting_point` | `text` |
| `starts_at` | `time` |
| `min_participants` | `integer not null default 1 check (min_participants >= 1)` |
| `max_participants` | `integer check (max_participants is null or max_participants >= 1)` |
| `min_age` | `integer check (min_age is null or min_age between 0 and 120)` |
| `max_age` | `integer check (max_age is null or max_age between 0 and 120)` |
| `requirements` | `text` |
| `availability` | `text not null default 'available' check (availability in ('available','on_request','seasonal','not_available'))` |
| `available_from` | `date` |
| `available_to` | `date` |
| `days_of_week` | `smallint[] not null default '{}'` |
| `cancellation` | `jsonb not null default '{}'::jsonb` |
| `photos` | `text[] not null default '{}'` |
| `photo_meta` | `jsonb not null default '{}'::jsonb` |
| `lead_time_hours` | `integer not null default 0 check (lead_time_hours >= 0)` |
| `daily_capacity` | `integer check (daily_capacity is null or daily_capacity >= 0)` |
| `is_active` | `boolean not null default true` |
| `sort_order` | `integer not null default 0` |
| `created_at` | `timestamptz not null default now()` |
| `updated_at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint experiences_participants check ( max_participants is null or max_participants >= min_participants )`
- `constraint experiences_ages check ( min_age is null or max_age is null or max_age >= min_age )`

## `hotel_config_audit`

Source: [`2026-08-23_who_changed_the_floor_price.sql`](../migrations/2026-08-23_who_changed_the_floor_price.sql). Declared in the listed DDL; verify it exists in the target project.

| Column | SQL declaration |
| --- | --- |
| `id` | `bigserial primary key` |
| `property_id` | `uuid not null references public.properties(id) on delete cascade` |
| `partner_id` | `uuid not null references public.partners(id) on delete cascade` |
| `entity` | `text not null` |
| `entity_id` | `uuid` |
| `entity_label` | `text` |
| `field` | `text not null` |
| `old_value` | `text` |
| `new_value` | `text` |
| `actor_member` | `uuid` |
| `actor_agent_action` | `uuid references public.agent_actions(id) on delete set null` |
| `actor_label` | `text not null default 'member'` |
| `at` | `timestamptz not null default now()` |

Table-level constraints:
- `constraint hotel_config_audit_actor check ( actor_member is not null or actor_agent_action is not null )`

## Definitions not covered here

This catalog extracts `CREATE TABLE` definitions only. It does not expand
columns introduced by `ALTER TABLE`, views, indexes, functions, triggers,
or policies. Review the migration sources for these. In particular, the
HostOS project has foundational tables not created by the current checked-in
base schema; see [Deploying StayOS](../DEPLOYING_STAYOS.md).
