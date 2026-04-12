# /update-memory

Rebuild the memory layers L2 and L1 from `src/` exports and `memory/L3/` service docs.

L3 is pre-built (per-repo/service docs) — this command does NOT touch it.
Inputs: `src/` (raw exports) + `memory/L3/` (service docs).
Outputs: `memory/L2/` (domain knowledge) + `memory/L1/` (navigation MOCs).

## Prerequisites

1. `src/` must be populated — run `/brain-sync` first if empty.
2. `memory/L3/` must be populated — run `/rebuild-services-docs` first if empty.

---

## Phase 1 — Discover Sources

Scan `src/` and `memory/L3/` to build a manifest of what's available.

```bash
# List all top-level source directories
ls src/

# For each, count files and note structure
for dir in src/*/; do
  echo "=== $dir ==="
  find "$dir" -type f | wc -l
  ls "$dir" | head -20
done

# Inventory L3 service docs
ls memory/L3/
```

Record all counts for the final digest.

---

## Phase 2 — L2 Rebuild (Domain Knowledge)

L2 files are topic summaries derived from `src/` and `memory/L3/`.
Don't maintain a fixed list — discover topics from what's available.

### 2a. Team files

Discover teams from whichever of these exist:
- Org config files in GitHub repos (terraform org yamls, IDP catalogs)
- `sources.md` comments (e.g. `# tium`, `# buktu` after repo URLs)
- Team-named folders in `src/clickup/` (e.g. `Docs <TeamName>/`)
- Linear project exports in `src/linear/`
- Service ownership in `memory/L3/` files

For each team discovered, write `memory/L2/team-<name>.md` containing:
- Members (from org config if available)
- Services (from L3 files whose repo is tagged to this team)
- Active projects (from `src/linear/` if available)
- Docs pointers (paths to team folders in clickup/confluence)

### 2b. Source-derived topics

For each top-level directory in `src/`, scan its structure and create/update L2 files:

| Source pattern | L2 file(s) to produce |
|----------------|----------------------|
| `src/<tool>/` with release notes folders | `releases.md` — count per period, highlight summaries |
| `src/<tool>/` with wiki/docs folders | One L2 per distinct wiki or doc collection — file counts, section inventory |
| `src/gdrive/` subfolders | One L2 per major folder (executive docs, proposals, updates, HR, etc.) |
| `src/confluence/` spaces | One L2 per space — page counts, section inventory |
| `src/metabase/` | `metabase.md`-style summary from index files |
| `src/medium/` or blog exports | `tone-of-voice.md` — analyze writing patterns |
| `src/github/` repos | `technologies.md` — aggregate stack info from L3 files |
| `src/personio/` or HR exports | Feed into ExCo/team files |

### 2c. L3-derived topics

Read `memory/L3/` service docs and extract cross-cutting summaries:
- `technologies.md` — aggregate tech stacks, languages, frameworks from all L3 files
- Feed service ownership and dependency info into team files
- Extract shared patterns (auth, infra, data) into domain-specific L2 files

### 2d. Cross-references

After all other L2 files are written, read them and extract any tables or timelines that span multiple domains into `cross-references.md`.

---

## Phase 3 — L1 Rebuild (Navigation MOCs)

L1 files are navigation maps derived from L2 and L3 inventory. Two categories:

### Source MOCs
For each source in `src/`, create/update a corresponding `memory/L1/<source>.md`:
- File counts by subfolder
- Links to every L2 file that draws from this source

### Cross-cutting MOCs
These synthesize across sources. Rebuild each by reading the L2 files it links to:

| L1 File | Derives from |
|---------|-------------|
| `teams.md` | All `team-*.md` L2 files + org config |
| `product-areas.md` | `releases.md` + team L2 files (group features by product area) |
| `business-domains.md` | ExCo + intranet + proposals L2 files (group by business function) |
| `data-model.md` | dbt/data repos in L3 + BigQuery metadata if available |
| `entities.md` | Full cross-source scan — anything appearing in 2+ sources gets an entry |
| `system-map.md` | Enumerate `.claude/agents/` and `.claude/skills/` |
| `skills.md` | Enumerate `.claude/skills/*/SKILL.md` |
| `hub.md` | **Last** — reads all other L1 files, builds the top-level nav with counts |

---

## Phase 4 — Verify

1. **Broken links**: grep all `[[wikilinks]]` in memory files, check each target exists
2. **Timestamps**: every `<!-- verified: -->` must have today's date
3. **Frontmatter**: every file's `updated:` = today
4. **Orphans**: memory files with no corresponding source → flag (don't delete)

---

## Phase 5 — Digest

Write `memory/L4/brain-sync/YYYY-MM-DD-rebuild.md` with:
- Source inventory table (discovered sources + file counts)
- L3 inventory (service docs count)
- Memory stats (files before/after per layer, created/updated/flagged)
- Changes summary
- Broken links
- Flagged for review

---

## Execution Order

```
Phase 1 (discover src + L3)
  → Phase 2 (L2 — domain knowledge from src + L3)
    → Phase 3 (L1 — navigation from L2 + L3)
      → Phase 4 (verify)
        → Phase 5 (digest)
```

## Rules

- **Discover, don't assume**: Scan directories to find what exists. Never hardcode team names, repo names, folder names, or file lists.
- **Source wins**: If a source contradicts existing memory, update memory.
- **L3 is read-only**: Never modify L3 files — they are inputs, not outputs.
- **Skip, don't fabricate**: If a source doesn't provide data for a section, use `<!-- TODO: source not available -->`.
- **Timestamp everything**: `<!-- verified: YYYY-MM-DD | source: ... -->` on every fact block.
- **Preserve `<!-- superseded: -->` markers**: Keep them even in a rebuild.
- **Conservative on entities**: 2+ source appearances required for `entities.md`.
- **Use `qmd query`** for semantic searches, `grep` for exact matches.
