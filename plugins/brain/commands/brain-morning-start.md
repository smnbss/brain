# /brain-morning-start

Daily bootstrap: update tools (brew, npm, gstack, python), sync brain sources, harvest yesterday's meeting notes, and prepare today's deep-dive and 1:1 agendas.

## Part 1 — Update tools & sync

1. **Brew update & upgrade** — run `brew update && brew upgrade` to update Homebrew and upgrade all installed formulae and casks.

2. **Global npm packages** — run `npm update -g` to update all globally installed npm packages (includes `qmd`).

3. **Gstack** — run `/gstack-upgrade` to update gstack to the latest version.

4. **Python packages** — run `uv sync --upgrade` to update the Python packages used by this repo.

5. **qmd reindex** — run `npx qmd update` (or `uv run qmd update` if using uv scripts) to rebuild the brain search index with any new content.

6. **Git pull** — run `git pull --rebase` to pull the latest brain changes.

7. **Report** — summarize what was updated, flag any errors or version bumps worth noting.

## Part 2 — Brain sync

Run `/brain-sync` to refresh L2 memory files from source exports and agent outputs. This keeps the brain's knowledge layer up to date with the latest data from ClickUp, Linear, GitHub, and other sources.

Run this as a background agent so Parts 3 and 4 can start in parallel.

## Part 3 — Harvest yesterday's meeting notes

Run the meeting harvester (see `.claude/skills/weroad-brain-sync/references/meetings.md`) for **yesterday** (the default range). This fetches calendar events, grabs Gemini notes and recording transcripts from Google Drive attachments, and produces a daily digest with decisions and action items.

Output goes to `src/gws/gmeet/YYYY/WNN/MM-DD/`.

## Part 4 — Prepare today's meetings

Run `/brain-prepare-my-day` to prepare agendas for all deep dives and 1:1s happening today. This fetches today's calendar, classifies events, and spawns the deep-dive and 1:1 agents in parallel.

Parts 2, 3, and 4 can all run in parallel — launch them at the same time.

## Part 5 — Final report

After all agents complete, print a combined summary:
```
Morning start complete:

Tools:
  - brew: <N> packages upgraded
  - npm/gstack/python: <status>

Brain sync:
  - <status — files refreshed, stale sources flagged>

Yesterday's meetings harvested:
  - <N> meetings processed → src/gws/gmeet/...

Today's prep:
  - <list of agendas generated with file paths>
```
