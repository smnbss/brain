# /prepare-my-day

Prepare agendas for all deep dives and 1:1s happening today. Fetches today's calendar, then spawns the deep-dive and 1:1 agents for each matching meeting.

## Steps

1. **Fetch today's calendar** — compute today's date boundaries in Europe/Rome (start of day → end of day), convert to UTC, and fetch events:

```bash
gws calendar events list --params '{
  "calendarId": "primary",
  "timeMin": "<TODAY_START_UTC>",
  "timeMax": "<TODAY_END_UTC>",
  "singleEvents": true,
  "orderBy": "startTime"
}'
```

2. **Classify events** — scan the results and sort into two buckets:

| Type | Match rule |
|------|-----------|
| Deep Dive | `summary` contains "Deep Dive" (case-insensitive) |
| 1:1 | `summary` contains "1:1" (case-insensitive), excluding "Prepare for 1:1s" |

Skip cancelled events (`status: "cancelled"`). Log any events that partially match but don't fit either bucket.

3. **Report the day's schedule** — print a quick summary of what was found before starting the agents:
```
Today's meetings to prepare:
- 11:00 — Deep Dive SAITAMA
- 14:00 — 1:1 Alex
- 16:00 — 1:1 Ryan
```

If no deep dives or 1:1s are found, report "No deep dives or 1:1s on the calendar today" and stop.

4. **Run deep-dive agents** — for each deep dive found, spawn an Agent running `.claude/agents/my-deep-dives/AGENT.md` with `LOOKAHEAD_DAYS: 1` (today only). If multiple deep dives exist, run them **in parallel** using concurrent Agent tool calls.

5. **Run 1:1 agents** — for each 1:1 found, spawn an Agent running `.claude/agents/my-one-on-one/AGENT.md` scoped to that specific person with `LOOKAHEAD_DAYS: 1`. If multiple 1:1s exist, run them **in parallel** using concurrent Agent tool calls.

   Steps 4 and 5 can run in parallel with each other — launch all agents at once.

6. **Report results** — after all agents complete, print a summary:
```
Prep complete:
✓ Deep Dive SAITAMA → outputs/agents/my-deep-dives/saitama.md
✓ 1:1 Alex → outputs/agents/my-one-on-one/alex.md
✓ 1:1 Ryan → outputs/agents/my-one-on-one/ryan.md
```

## Agent references

- Deep dives: `.claude/agents/my-deep-dives/AGENT.md` — one file per team in `outputs/agents/my-deep-dives/`
- 1:1s: `.claude/agents/my-one-on-one/AGENT.md` — one file per person in `outputs/agents/my-one-on-one/`

## When to use

Run at the start of each working day to prepare all meeting agendas at once.
