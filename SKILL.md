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
  [--method  GET|POST|PUT|DELETE] \
  [--scope   read|write|update|delete] \
  [--payload <json>]
```

- **endpoint** — Google Calendar API path (relative to `/calendar/v3`)
- **intent** — why Claude is making this call (5–10 words, reflects the user's goal)
- **method** — defaults to `GET`; set `POST`/`PUT`/`DELETE` for mutations
- **scope** — inferred from method if omitted
- **payload** — JSON body for POST/PUT requests

## Key endpoints

| Action | Method | Endpoint |
|--------|--------|----------|
| List calendars | GET | `/users/me/calendarList` |
| List events | GET | `/calendars/{calendarId}/events` |
| Get event | GET | `/calendars/{calendarId}/events/{eventId}` |
| Create event | POST | `/calendars/{calendarId}/events` |
| Update event | PUT | `/calendars/{calendarId}/events/{eventId}` |
| Delete event | DELETE | `/calendars/{calendarId}/events/{eventId}` |
| Check free/busy | POST | `/freeBusy` |
| Quick add event | POST | `/calendars/{calendarId}/events/quickAdd` |

Use `primary` as `{calendarId}` for the user's default calendar.

## Token & scope

`OKORO_SERVICE_TOKEN` must have at least the required scope level:
`read` < `write` < `update` < `delete`

## Intent

Always pass `--intent` with the user's actual reason — not a description of the API call.
