---
name: brain-prepare-my-deep-dives
description: >
  Prepare deep-dive meeting agendas by fetching calendar events and active Linear projects
  for each team. Use when the user says "prep deep dive", "prepare deep dives", "deep dive agenda",
  or asks to prepare for a specific team's deep dive. Also triggered by /prepare-my-day for
  calendar events containing "Deep Dive".
---

# Deep Dive Prep Agent

Prepare deep-dive meeting files by fetching upcoming calendar events and active Linear projects for each team. Output: one file per team in `outputs/agents/my-deep-dives/`.

## Configuration

```
TIMEZONE: Europe/Rome
OUTPUT_DIR: outputs/agents/my-deep-dives
LOOKAHEAD_DAYS: 5
TODAY: (compute dynamically)
```

### Calendar-to-Linear team mapping

The team mapping lives in `references/deep-dive-teams.md` relative to this skill. Read it at the start of every run.

If the file does not exist, copy it from the skill's `references/` directory and tell the user to configure it before proceeding.

The calendar event summary contains "Deep Dive" plus a team name. Extract the team slug and map it to one or more Linear team names using the mapping file.

---

## Step 1 — Fetch upcoming deep-dive events

Compute the time window: from now to +LOOKAHEAD_DAYS (midnight Europe/Rome, converted to UTC).

```bash
gws calendar events list --params '{
  "calendarId": "primary",
  "timeMin": "<NOW_UTC>",
  "timeMax": "<END_UTC>",
  "singleEvents": true,
  "orderBy": "startTime",
  "q": "deep dive"
}'
```

Filter results:
1. Keep only events whose `summary` contains "Deep Dive" (case-insensitive)
2. Skip cancelled events (`status: "cancelled"`)
3. Extract for each event:
   - `summary` (to derive team slug)
   - `start.dateTime` (meeting date and time)
   - `attendees[]` (names/emails)
   - `description` (contains ODG/Roadmap/Deck links — preserve these)

**Team slug extraction:** Strip "Deep Dive", " - ", leading/trailing whitespace from the summary. Lowercase the remainder. Match against the mapping file. Examples:
- `"SAITAMA - Deep Dive"` → `saitama`
- `"DEVOPS & IT - Deep Dive"` → `devops`
- `"Deep Dive Tech"` → `tech`
- `"DATA - Deep Dive"` → `data`

---

## Step 2 — Fetch Linear projects for each team

For each team in the mapping, query Linear for active projects (not completed, not cancelled):

Use the `list_projects` Linear MCP tool:
- `team`: the Linear team name from the mapping
- `limit`: 50

Do this for **each Linear team** in the mapping. If the mapping has multiple teams (e.g., DEVOPS + IT), query each and merge results, deduplicating by project ID.

**Collect for each project:**
- `name`
- `status.name` (Backlog, Planned, In Progress, Completed, Cancelled)
- `lead.name` (or "— (no lead)" if null)
- `targetDate` (or "—" if null)
- `priority.name` (Urgent, High, Medium, Low, or null)
- `updatedAt`
- `startDate`
- `labels`
- `createdAt`

**Filter out:**
- Projects with status "Completed" or "Cancelled"

**Compute flags for each project:**
- `OVERDUE`: targetDate is in the past (before today) AND status is not Completed/Cancelled
- `At Risk`: project has priority "Urgent" and is not In Progress, OR it is OVERDUE
- `No update Nd`: updatedAt is more than 14 days before today — show as "no update {N}d"
- `No target`: targetDate is null AND status is "In Progress" or "Planned"
- `NEW`: createdAt is within the last 14 days — show as "NEW ({date})"

A project can have multiple flags, comma-separated.

**Sort order:** In Progress first (by targetDate ascending, nulls last), then Planned (by targetDate ascending), then Backlog.

---

## Step 3 — Generate the agenda

For each deep dive, analyze the project data and generate a **pointed, to-the-point agenda** with sharp questions. The agenda is what makes these files useful — it should surface what needs attention.

### Agenda generation rules

**Section 1: Items due before the next deep dive** (targetDate <= today + 7 days AND status In Progress or Planned)
- For each: state what's expected and ask if it will ship on time
- If already overdue: ask what blocked it and what the new date is

**Section 2: Overdue items** (targetDate < today)
- Group all overdue projects
- For each: how many days/weeks overdue, last update date, direct question about what's blocking

**Section 3: At-risk / stale items** (no update in 14+ days, or flagged At Risk)
- For each: flag the staleness, ask for status

**Section 4: Notable upcoming work** (next 2-4 weeks, In Progress or Planned)
- Brief list of what's coming, any cross-team dependencies to call out

**Section 5: Capacity / ownership gaps**
- Projects with no lead assigned
- If one person leads everything, flag it
- Cross-team projects that touch this team

**Tone:** Direct, questioning, concise. No filler. Write like a CTO prepping for a 30-minute check-in. Examples:
- "3 weeks overdue. What's blocking?"
- "No update in 14 days. Still on track?"
- "Target is Apr 17 but still in Backlog. Realistic?"
- "One person leads ALL 10 projects. Capacity concern?"

### "Your topics" section

Always include an empty `## Your topics` section at the end of the agenda (before the Linear table) for manual additions.

---

## Step 4 — Write the output file

Write one file per team to `outputs/agents/my-deep-dives/<slug>.md`.

**If the file already exists, overwrite it completely** — each run produces a fresh file.

### Output format

```markdown
# <TEAM> Deep Dive — W<NN> (<Day>, <Mon DD>, <HH:MM>)
<!-- generated: YYYY-MM-DD -->

## <Section title>
- **<Project name>** (<target date>) — <status/context>. <Pointed question>

## <Next section>
...

## Your topics


---

## Linear Projects — <TEAM> Active

| Project | Status | Lead | Target | Flag |
|---------|--------|------|--------|------|
| Project name | In Progress | Lead Name | Apr 30 | On Track |
| Another project | Planned | Lead Name | — | No target |
| Overdue thing | In Progress | Lead Name | Mar 31 | OVERDUE |
```

**Week number:** Use ISO 8601 week numbering (Monday start).

**Day/time format:** Use the event's `start.dateTime` converted to Europe/Rome. Show as `Day, Mon DD, HH:MM` (e.g., `Thu, Apr 9, 11:00`).

---

## Step 5 — Report results

Print a summary:
- How many deep dives found in the next LOOKAHEAD_DAYS
- For each: team name, date/time, number of active projects, number of flags (overdue/at-risk/stale)
- Path to each generated file

---

## Running

**Default (next 5 days):**
```
/my-deep-dives
```

**Custom lookahead:**
```
/my-deep-dives --days 10
```

**Single team:**
```
/my-deep-dives saitama
```

Override LOOKAHEAD_DAYS or filter to a single team accordingly.
