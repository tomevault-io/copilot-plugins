## project-3-real-time-ai-sales-lead-intelligence-engine

> Celery async task rules for AI pipeline jobs — transcription, analysis, prediction, notifications


Celery stack:
- Celery with Redis broker (`CELERY_BROKER_URL=redis://...`)
- Redis result backend (`CELERY_RESULT_BACKEND=redis://...`)
- Task queues: `transcription`, `analysis`, `prediction`, `notifications` (separate queues per domain)

Task organization:
- `tasks/transcription_tasks.py` — Post-call batch Whisper transcription
- `tasks/analysis_tasks.py`      — Post-call GPT-4o deep analysis (sentiment, topics, objections, scorecard)
- `tasks/prediction_tasks.py`    — Deal outcome prediction (re-run after each call)
- `tasks/notification_tasks.py`  — Async email/in-app notification dispatch
- `tasks/celery_app.py`          — Celery app factory + task routing configuration

Task rules:
- All AI pipeline tasks (transcription, analysis, prediction) must be async-safe and idempotent.
- Idempotency: if a task runs twice for the same `call_id`, it must not create duplicate records — use upsert or check-before-insert.
- Tasks must not block on network I/O — use `asyncio.run()` within a Celery task to call async services, or use `celery[gevent]`/`celery[eventlet]` if the team prefers cooperative concurrency.
- Task failures must be logged with the `call_id`, `tenant_id`, and error details for debugging.
- Use Celery task retries with exponential backoff for transient failures (OpenAI rate limits, S3 timeouts):
  ```python
  @app.task(bind=True, max_retries=3, default_retry_delay=30)
  def transcribe_call(self, call_id: str): ...
  ```
- Do not share SQLAlchemy async sessions across task boundaries — create a new session per task.

Post-call pipeline order (enforced):
1. Twilio recording callback → `transcription_tasks.transcribe_call(call_id)` dispatched
2. Transcription complete → `analysis_tasks.analyze_call(call_id)` dispatched
3. Analysis complete → `prediction_tasks.predict_deal(lead_id, call_id)` dispatched
4. Prediction stored → `notification_tasks.notify_deal_update(lead_id, tenant_id)` dispatched (if thresholds met)

Queue routing:
- Route transcription tasks to `transcription` queue — can scale workers independently.
- Route analysis tasks to `analysis` queue — GPT-4o calls are expensive; limit concurrency.
- Route prediction tasks to `prediction` queue.
- Route notification tasks to `notifications` queue.

Do not:
- Run AI pipeline tasks synchronously in HTTP request handlers — always dispatch to Celery.
- Mix task logic with service business logic — tasks should call services, not implement logic directly.
- Ignore task failures silently — always log and optionally retry.
- Use `task.apply_async()` without a queue name — always specify the correct queue.
- Create circular task dependencies (e.g. prediction task calling transcription task).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/euronone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
