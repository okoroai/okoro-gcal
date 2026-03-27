---
name: gcal
description: "Manage Google Calendar events, check availability, and schedule meetings through the Okoro proxy."
version: 1.0.0

# Claude Code fields
argument-hint: "--endpoint /<resource> --intent \"describe action\""
allowed-tools: Bash

# OpenClaw fields
metadata:
  openclaw:
    requires:
      env:
        - OKORO_SERVICE_TOKEN
      bins:
        - curl
        - jq
    primaryEnv: OKORO_SERVICE_TOKEN
    homepage: https://okoro.ai
    os:
      - darwin
      - linux
---

You have access to the Google Calendar skill via the Okoro proxy. Use `scripts/gcal.sh` for all
Google Calendar operations. The script caches the session token and refreshes it automatically on expiry.

## Usage

```bash
skills/gcal/scripts/gcal.sh \
  --endpoint <path> \
  --intent   <reason> \
  [--method  GET|POST|PATCH|PUT|DELETE] \
  [--scope   read|write|update|delete|all] \
  [--payload <json>]
```

- **endpoint** — Google Calendar API path including query parameters (e.g. `/v3/calendars/primary/events?maxResults=10&timeMin=2026-01-01T00:00:00Z`)
- **intent** — the session intent: the user's overall goal for this conversation (not a description of the API call)
- **method** — defaults to `GET`; set `POST`/`PATCH`/`PUT`/`DELETE` for mutations
- **scope** — inferred from method if omitted (`GET`→read, `POST`→write, `PATCH`/`PUT`→update, `DELETE`→delete). **Always pass `--scope all` explicitly for `freeBusy`** — auto-inference gives `write` for POST.
- **payload** — JSON body for POST/PATCH/PUT requests only. **Never use `--payload` with GET or HEAD** — pass filters and options as query parameters in `--endpoint` instead.

## Key endpoints

| Action | Method | Endpoint | Min scope |
|--------|--------|----------|-----------|
| List calendars | GET | `/v3/users/me/calendarList` | `read` |
| List events | GET | `/v3/calendars/{calendarId}/events` | `read` |
| Get event | GET | `/v3/calendars/{calendarId}/events/{eventId}` | `read` |
| Create event | POST | `/v3/calendars/{calendarId}/events` | `write` |
| Quick add event | POST | `/v3/calendars/{calendarId}/events/quickAdd?text=<url-encoded text>` | `write` |
| Update event (partial) | PATCH | `/v3/calendars/{calendarId}/events/{eventId}` | `update` |
| Replace event (full) | PUT | `/v3/calendars/{calendarId}/events/{eventId}` | `update` |
| Delete event | DELETE | `/v3/calendars/{calendarId}/events/{eventId}` | `delete` |
| Check free/busy | POST | `/v3/freeBusy` | `all` ⚠ pass `--scope all` |

Use `primary` as `{calendarId}` for the user's default calendar.

> **PATCH vs PUT:** Use `PATCH` for partial updates (only send fields you want to change). Use `PUT` only when replacing the entire event resource — omitted fields will be erased. Prefer PATCH.

> **quickAdd:** The `text` parameter is a **query parameter**, not a payload field. Do not use `--payload`. Example: `--endpoint "/v3/calendars/primary/events/quickAdd?text=Lunch+with+Bob+tomorrow+at+noon"`.

> ⚠ POST auto-infers `write` scope. `freeBusy` requires `all` — pass `--scope all` explicitly or the proxy will reject the request with 403.

## Token & scope

`OKORO_SERVICE_TOKEN` must have at least the required scope level:
`read` < `write` < `update` < `delete` < `all`

**Scope auto-inference:** `GET`→`read`, `POST`→`write`, `PATCH`/`PUT`→`update`, `DELETE`→`delete`. The `freeBusy` POST endpoint requires `all` scope — you must pass `--scope all` explicitly.

## Typical workflows

**List upcoming events:**
```bash
skills/gcal/scripts/gcal.sh \
  --endpoint "/v3/calendars/primary/events?maxResults=10&orderBy=startTime&singleEvents=true&timeMin=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  --intent "show upcoming calendar events"
```

**Create an event:**
```bash
# start and end dateTime are required; summary (title) is strongly recommended
skills/gcal/scripts/gcal.sh --method POST \
  --endpoint /v3/calendars/primary/events \
  --intent "schedule team meeting" \
  --payload '{
    "summary": "Team sync",
    "start": {"dateTime": "2026-04-01T10:00:00+00:00"},
    "end":   {"dateTime": "2026-04-01T11:00:00+00:00"}
  }'
```

**Quick-add an event (natural language):**
```bash
# text goes as a query parameter — no --payload
skills/gcal/scripts/gcal.sh --method POST \
  --endpoint "/v3/calendars/primary/events/quickAdd?text=Lunch+with+Bob+tomorrow+at+noon" \
  --intent "add lunch event from user message"
```

**Patch an event (partial update):**
```bash
# Only send the fields to change — other fields are preserved
skills/gcal/scripts/gcal.sh --method PATCH \
  --endpoint /v3/calendars/primary/events/<eventId> \
  --intent "reschedule meeting" \
  --payload '{"start":{"dateTime":"2026-04-02T10:00:00+00:00"},"end":{"dateTime":"2026-04-02T11:00:00+00:00"}}'
```

**Check free/busy:**
```bash
# Must pass --scope all — POST alone would only get write scope
skills/gcal/scripts/gcal.sh --method POST --scope all \
  --endpoint /v3/freeBusy \
  --intent "find free slot for team meeting" \
  --payload '{
    "timeMin": "2026-04-01T00:00:00Z",
    "timeMax": "2026-04-07T00:00:00Z",
    "items": [{"id": "primary"}]
  }'
```

## Intent

`--intent` is the **session intent** — the user's overall goal for this conversation, not a description of the individual API call. It is logged by the proxy as the audit reason for every token issued in this session. Pass the same value for every call you make within a single user request.

```
--intent "find a free slot for a team meeting next week"   ✓  (why the user asked)
--intent "query freeBusy endpoint"                          ✗  (describes the API call)
--intent "get calendar events"                              ✗  (too vague, still call-level)
```
