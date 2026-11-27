# Background Workers and Schedules

TravBuddy uses Celery with Redis for asynchronous work and scheduled
maintenance. The API can start without a worker, but queued tasks will not
complete. Check `/api/v1/health` for broker and worker readiness.

## Worker tasks

| Task | Queue | Purpose |
| --- | --- | --- |
| `travbuddy.analyze_reel` | default | Run the Reel download, scene, frame, vision, and persistence pipeline |
| `travbuddy.reco.embed_hotel` | default | Generate/update a hotel embedding |
| `travbuddy.reco.profile_hotel` | default | Generate/update hotel profile content |
| `travbuddy.reco.embed_and_profile` | default | Combine hotel embedding and profile work |
| `travbuddy.reco.refresh_expected_price` | default | Refresh recommendation expected-price aggregates |
| `travbuddy.reco.refresh_destination_priors` | default | Refresh destination priors |
| `travbuddy.reco.fit_centroids` | default | Refit recommendation personality centroids |
| `travbuddy.partners.revert_intro_commission` | default | Revert expired partner introductory commission rates |
| `travbuddy.messaging.process_conversation` | `hotelos` | Process a persisted guest conversation |
| `travbuddy.messaging.sweep_waiting_guests` | `hotelos` | Find conversations waiting for human follow-up |
| `travbuddy.pms.reconcile_inventory` | `hotelos` | Report hotel inventory drift |
| `travbuddy.marketing.run_campaign` | `hotelos` | Run an eligible hotel marketing campaign |

Task definitions are in `app/workers/`; `app/workers/celery_app.py` registers
the application, imports task modules, configures routing, and sets a 24-hour
result expiry.

## Queue isolation

- `travbuddy.messaging.*`, `travbuddy.marketing.*`, and `travbuddy.pms.*` route
  to the `hotelos` queue.
- Other tasks use Celery's default queue.
- HostOS workers should consume `hotelos`; the marketplace worker should not
  consume hotel tasks against the travel database.
- Both deployments need the intended broker/database configuration. Queue names
  alone do not isolate workers that are connected to different Redis instances.

## Beat schedules

`CELERY_BEAT_ROLE` selects which schedule a beat process owns:

| Role | Scheduled work |
| --- | --- |
| `marketplace` | Expected-price refresh daily; destination-prior refresh daily; centroid fitting weekly; partner commission reversion hourly |
| `stayos` | Waiting-guest sweep every 5 minutes; inventory-drift report hourly |
| `all` (default) | Both schedule sets |

Run only one beat instance per intended schedule/broker. Running multiple
instances can enqueue duplicate scheduled work. Keep role and broker aligned
with the database the tasks are allowed to access.

## Local commands

Run Redis separately, then start the worker from the repository root:

```powershell
redis-server
.\.venv\Scripts\python.exe -m celery -A app.workers.celery_app.celery_app worker --loglevel=info
```

For the scheduler:

```powershell
$env:CELERY_BEAT_ROLE = "marketplace"
.\.venv\Scripts\python.exe -m celery -A app.workers.celery_app.celery_app beat --loglevel=info
```

For HostOS, configure the HostOS environment and consume the `hotelos` queue.
See [Deploying StayOS](../DEPLOYING_STAYOS.md) and
[integration.md](../integration.md) before running workers.
