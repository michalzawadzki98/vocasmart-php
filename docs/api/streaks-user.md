# User streaks API contract

Status: **proposed / needs owner confirmation**  
Consumers: VocaSmart mobile app (`GET` via `/api/v2/gameification/streaks/user`)  
Related frontend PR: streak Celebration sheet (uses this payload to decide “played today”).

## Endpoint

| | |
|---|---|
| Method | `GET` |
| Path | `/api/v2/gameification/streaks/user` |
| Auth | Authenticated user (same as other v2 user resources) |

> Note: path spelling `gameification` matches the mobile client today. Prefer keeping the path stable; if renaming to `gamification`, coordinate a client release.

## Response shape (current client expectation)

Envelope matches other v2 resources (`data` wrapper). Payload fields consumed by the app:

| Field | Type (client) | Meaning assumed by client |
|---|---|---|
| `user_id` | number / string | Owner of the streak |
| `current_streak` | number | Consecutive active days; `0` = no active streak |
| `longest_streak` | number | All-time best consecutive days |
| `last_activity_date` | string | Last day that counted toward the streak |

### Example (illustrative)

```json
{
  "data": {
    "user_id": 42,
    "current_streak": 5,
    "longest_streak": 12,
    "last_activity_date": "2026-07-27"
  }
}
```

## Open questions (block production “today secured” semantics)

Please confirm with examples from production / staging:

1. **Format of `last_activity_date`** — `YYYY-MM-DD` only, ISO-8601 datetime, or other?
2. **Timezone / calendar day** used for increment and reset (UTC, server TZ, user profile TZ)?
3. **Increment rule** — which user actions count as “activity” for a day?
4. **Reset rule** — miss one calendar day vs rolling 24h window?
5. **Preferred client approach** —
   - A) Client compares `last_activity_date` to “today” in a documented TZ, or
   - B) API adds an explicit boolean (e.g. `played_today` / `today_secured`)?

## Interim client behaviour (until confirmed)

Mobile treats “played today” as:

- `current_streak > 0`, and
- normalized `last_activity_date` equals today’s **device-local** calendar date (`YYYY-MM-DD`, or date part of an ISO string in local TZ).

This is an assumption for UX only — backend owners should either ratify it or publish the authoritative rule / flag above.

## Suggested follow-up implementation (out of scope for this docs PR)

If streaks are not yet implemented in this repository:

1. Persist per-user `current_streak`, `longest_streak`, `last_activity_date`.
2. Expose `GET /api/v2/gameification/streaks/user` matching the table above.
3. Document day boundary + activity sources in this file and regenerate OpenAPI (`composer open-api`).
4. Optionally add `played_today: boolean` computed server-side so clients do not guess TZ.
