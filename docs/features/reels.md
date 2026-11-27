# Reel-to-Location Intelligence

## User flow

1. Submit an Instagram Reel URL.
2. Receive a job identifier while the pipeline runs asynchronously.
3. Poll status or subscribe to server-sent progress events.
4. Retrieve the completed location timeline and optionally save or attach it to
   a traveller trip.

## API surface

- `POST /api/v1/reels/analyze` — enqueue analysis.
- `GET /api/v1/reels/jobs/{job_id}` — inspect job status.
- `GET /api/v1/reels/jobs/{job_id}/result` — retrieve the result.
- `GET /api/v1/reels/jobs/{job_id}/stream` — stream progress using SSE.
- `GET /api/v1/reels/saved` — list saved Reels.
- `POST /api/v1/reels/{reel_id}/like` — toggle a like.
- `/api/v1/trips/{trip_id}/reels*` and `/saved-reels*` — attach or manage Reel
  references on trips.

## Processing stages

The pipeline modules in `app/pipeline/` implement:

1. Video acquisition using `yt-dlp` and supported fallback logic.
2. Scene boundary detection using PySceneDetect.
3. Keyframe extraction using OpenCV.
4. Perceptual-hash deduplication to reduce redundant frames.
5. Gemini Vision location extraction.
6. Timeline assembly and location merging.
7. Optional Google Places enrichment and Supabase persistence.

The job task is `travbuddy.analyze_reel`; Celery and Redis must be available for
the asynchronous pipeline. Google Cloud Storage is optional for deployments
that persist media assets.

## Limits and configuration

The pipeline has configurable duration, size, frame, location, and confidence
limits in `app/config.py` and `.env.example`. `INSTAGRAM_PROXY_URL` is an
optional proxy setting. Use the actual environment values and provider
constraints; do not assume public Reel URLs are always downloadable.

## Implementation and tests

- API: `app/api/v1/reels/`
- Pipeline: `app/pipeline/`
- Worker: `app/workers/celery_app.py`
- Related tests: `tests/test_downloader.py`, `tests/test_scene_detector.py`,
  and pipeline-related tests.

The processing architecture is also summarized in the root [Features.md](../../Features.md).
