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

You have access to the Gcal skill via the Okoro proxy. Use `scripts/gcal.sh` for all
Gcal operations. The script caches the session token and refreshes it automatically on expiry.

## Usage

```bash
skills/gcal/scripts/gcal.sh \
  --endpoint <path> \
  --intent   <reason> \
  [--method  GET|POST|PUT|DELETE] \
  [--scope   read|write|update|delete] \
  [--payload <json>]
```

- **endpoint** — Gcal API path
- **intent** — why Claude is making this call (5–10 words, reflects the user's goal)
- **method** — defaults to `GET`; set `POST`/`PUT`/`DELETE` for mutations
- **scope** — inferred from method if omitted
- **payload** — JSON body for POST/PUT requests

## Key endpoints

<!-- TODO: document key API endpoints for Gcal -->

| Action | Method | Endpoint |
|--------|--------|----------|
| Example | GET | `/example` |

## Token & scope

`OKORO_SERVICE_TOKEN` must have at least the required scope level:
`read` < `write` < `update` < `delete`

## Intent

Always pass `--intent` with the user's actual reason — not a description of the API call.
