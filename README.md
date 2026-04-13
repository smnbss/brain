# brain

[![skills.sh](https://img.shields.io/badge/skills.sh-compatible-blue)](https://skills.sh)

AI agent skills for building and syncing a knowledge brain. One repository, every tool.

## Install

```bash
npx skills add smnbss/brain
```

Works with Claude Code, Gemini CLI, Codex CLI, Cursor, Copilot, and 30+ other tools.

Install specific skills or target specific tools:

```bash
npx skills add smnbss/brain --skill brain-pull-sources -a cursor -a codex
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `brain-pull-sources` | Export all external sources — ClickUp, Confluence, GDrive, Linear, GitHub, Medium, Metabase |
| `brain-rebuild-services` | Generate deep service `.AGENT.MD` docs from cloned GitHub repos |
| `brain-rebuild-memory` | Rebuild L2 (domain knowledge) and L1 (navigation MOCs) from `src/` + `outputs/services/` |
| `brain-git-sync` | Commit and push all brain changes |
| `brain-pull-my-meeting-notes` | Harvest meeting notes, recordings, and transcripts from Google Calendar and Drive |
| `brain-prepare-my-one-on-one` | Prepare 1:1 meeting agendas from Linear, brain context, and previous agendas |
| `brain-prepare-my-deep-dives` | Prepare deep-dive meeting agendas from Linear project data per team |
| `brain-morning-start` | Daily bootstrap: update tools, sync brain, harvest meeting notes, prepare agendas |
| `brain-prepare-my-day` | Fetch today's calendar and generate deep-dive and 1:1 agendas in parallel |
| `brain-push-reports` | Push latest agent report outputs to ClickUp document pages |
| `brain-weekly-review` | Compile weekly summary from Workflowy, X posts, and Linear updates |

## Required Environment

Create `.env.local` in your brain project root:

```bash
CLICKUP_TOKEN=...
CONFLUENCE_EMAIL=...
CONFLUENCE_BASE_URL=https://weroad.atlassian.net/wiki
CONFLUENCE_TOKEN=...
LINEAR_TOKEN=...
METABASE_URL=...
METABASE_API_KEY=...
GITHUB_TOKEN=...       # optional, for private repos
GCP_PROJECT_ID=...
```

## Quick Start

1. **Populate sources** — invoke the `brain-pull-sources` skill to export all external data into `src/`
2. **Build service docs** — invoke the `brain-rebuild-services` skill to generate per-repo architecture docs
3. **Build memory** — invoke the `brain-rebuild-memory` skill to synthesize L2 and L1 navigation maps
4. **Save** — invoke the `brain-git-sync` skill to commit and push

## Architecture

```
smnbss/brain/
├── skills/
│   ├── brain-pull-sources/
│   │   ├── SKILL.md
│   │   ├── bin/
│   │   ├── utils/
│   │   └── references/
│   ├── brain-rebuild-services/
│   │   └── SKILL.md
│   ├── brain-rebuild-memory/
│   │   └── SKILL.md
│   ├── brain-git-sync/
│   │   └── SKILL.md
│   ├── brain-pull-my-meeting-notes/
│   │   └── SKILL.md
│   ├── brain-prepare-my-one-on-one/
│   │   └── SKILL.md
│   ├── brain-prepare-my-deep-dives/
│   │   └── SKILL.md
│   ├── brain-morning-start/
│   │   └── SKILL.md
│   ├── brain-prepare-my-day/
│   │   └── SKILL.md
│   ├── brain-push-reports/
│   │   └── SKILL.md
│   └── brain-weekly-review/
│       └── SKILL.md
├── CLAUDE.md
├── AGENTS.md -> CLAUDE.md
└── README.md
```

**Design decisions:**
- Skills follow the [Agent Skills open standard](https://skills.sh/) (`SKILL.md` with YAML frontmatter) — compatible with 40+ tools
- Single distribution path: `npx skills add` handles all CLIs
- No CLI-specific packaging — one format, works everywhere
