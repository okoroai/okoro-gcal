---
name: gcal
description: "Manage Google Calendar events, check availability, and schedule meetings through the Okoro proxy."
tags: [calendar, productivity, google]
version: 1.0.0
repo: okoroai/okoro-gcal
---

# gcal

Manage Google Calendar events, check availability, and schedule meetings through the Okoro proxy.

All API calls go through the [okoro proxy](https://okoro.ai), which signs requests, enforces permission scopes, and writes an audit trail for every action your agent takes.

## Requirements

| Variable | Description |
|---|---|
| `OKORO_SERVICE_TOKEN` | Service token from the [okoro dashboard](https://hub.okoro.ai/docs/get-token) (`svc_...`) |

[How to get your token →](https://hub.okoro.ai/docs/get-token)

## How it works

Rather than giving your agent raw Gcal credentials, you configure a **service token** once. When the skill runs, it exchanges that token for a short-lived **operation token** scoped to exactly the action being performed. The okoro proxy signs the request, forwards it to Gcal, and records the intent in an audit log.

[Learn more about how okoro works →](https://okoro.ai/how-it-works)

## What your agent can do

<!-- TODO: list operations for this provider -->

## License

MIT — see [LICENSE](./LICENSE).
