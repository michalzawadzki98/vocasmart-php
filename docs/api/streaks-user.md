# User streaks API contract

Status: **needs owner confirmation** (endpoint already consumed by mobile)  
Consumers: VocaSmart mobile app — `USER_STREAKS` → `GET /api/v2/gameification/streaks/user`  
Related frontend work: streak Celebration sheet (uses this payload to decide “played today”).

> This doc does **not** request creating a new endpoint. The client already calls this path. We need the authoritative semantics confirmed (and optionally a `played_today` flag).

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

### Example (illustrative — replace with a real staging/prod sample)

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

## Optional follow-up (only if useful)

- Add `played_today: boolean` (or `today_secured`) computed server-side so clients do not guess TZ.
- Mirror this contract into OpenAPI (`composer open-api`) once semantics are locked.
