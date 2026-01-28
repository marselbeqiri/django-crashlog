# Django Stacktrace

Install from PyPI:

```bash
pip install django-stacktrace
```

A reusable Django app that captures exceptions with request context and stores
them in the database, similar to a lightweight Sentry. It is best suited for
early-stage apps that do not yet need to invest in a full error tracking tool;
for production-grade monitoring, Sentry is recommended.

## Quick start

1. Add the app:
   - `django_stacktrace` to `INSTALLED_APPS`
2. Add middleware (signal fallback is disabled automatically when middleware is enabled):
   - `django_stacktrace.middleware.StacktraceMiddleware` in `MIDDLEWARE`
3. Apply migrations:
   - `python manage.py migrate`
4. Trigger a crash and inspect `Crash events` in the Django admin.

## Manual capture

You can capture exceptions directly without middleware:

```python
from django_stacktrace.event_store import store_crash_event

try:
    1 / 0
except Exception as exc:
    store_crash_event(exc=exc)
```

## Settings

All settings live under a single `STACKTRACE` dictionary. Each key is optional.

```python
STACKTRACE = {
    "ENABLED": True,
    "SAMPLE_RATE": 1.0,
    "RATE_LIMIT": 0,
    "CAPTURE_HEADERS": True,
    "CAPTURE_BODY": False,
    "MAX_PAYLOAD_BYTES": 64 * 1024,
    "REDACT_FIELDS": ["password", "token"],
    "REDACT_HEADERS": ["authorization", "cookie"],
    "USER_FIELD": "username",
}
```

Available keys:

- `ENABLED` (default `True`)
- `SAMPLE_RATE` (default `1.0`, 0-1 sampling probability)
- `RATE_LIMIT` (default `0`, per-minute cap; `0` disables)
- `CAPTURE_HEADERS` (default `True`)
- `CAPTURE_BODY` (default `False`)
- `MAX_PAYLOAD_BYTES` (default `65536`)
- `REDACT_FIELDS` (request keys to mask)
- `REDACT_HEADERS` (header keys to mask)
- `USER_FIELD` (user attribute for display name)

## Settings reference

`ENABLED`

- Type: `bool`
- Purpose: Enable or disable crash capturing entirely.
- Example: `"ENABLED": False`

`SAMPLE_RATE`

- Type: `float` between `0.0` and `1.0`
- Purpose: Probabilistic sampling for high-volume apps.
- Example: `"SAMPLE_RATE": 0.1`

`RATE_LIMIT`

- Type: `int`
- Purpose: Per-minute cap; `0` disables rate limiting.
- Example: `"RATE_LIMIT": 120`

`CAPTURE_HEADERS`

- Type: `bool`
- Purpose: Include request headers in the captured event.
- Example: `"CAPTURE_HEADERS": True`

`CAPTURE_BODY`

- Type: `bool`
- Purpose: Include request body in the captured event.
- Example: `"CAPTURE_BODY": False`

`MAX_PAYLOAD_BYTES`

- Type: `int`
- Purpose: Max bytes kept from request body.
- Example: `"MAX_PAYLOAD_BYTES": 65536`

`REDACT_FIELDS`

- Type: `list[str]` or `set[str]`
- Purpose: Request data keys to mask before storage.
- Example: `"REDACT_FIELDS": ["password", "token"]`

`REDACT_HEADERS`

- Type: `list[str]` or `set[str]`
- Purpose: Header keys to mask before storage.
- Example: `"REDACT_HEADERS": ["authorization", "cookie"]`

`USER_FIELD`

- Type: `str`
- Purpose: User attribute used for display name in the event.
- Example: `"USER_FIELD": "username"`

## Data model

Events are stored in `django_stacktrace.CrashEvent` with indexed fields like level,
logger, error type, request path, user identifier, and traceback hash, plus a
JSON payload for full context.

## Backlog

- Async storage path (Celery/queue support)
- Logging integration for `logging` exceptions
- Rate-limit options (per-user/IP limits, key prefix)
- Body size controls (per-content-type limits, truncation)
- Filtering hooks (ignore exceptions or URLs)
- Admin UI improvements (grouping, search, trends)
- Export and retention tools (cleanup, CSV export)