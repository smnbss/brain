# brain

[![skills.sh](https://img.shields.io/badge/skills.sh-compatible-blue)](https://skills.sh)

AI agent skills for building and syncing the WeRoad knowledge brain. One repository, every tool.

## How to Use

### Claude Code (recommended — zero friction)

Add this marketplace to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "smnbss-brain": {
      "source": {
        "source": "github",
        "repo": "smnbss/brain"
      },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "brain@smnbss-brain": true
  }
}
```

Claude will prompt you to trust the marketplace on first open. After that, skills auto-update with every conversation.

**Note**: Plugins can also be managed via the Claude TUI via the `/plugin` shortcut.

### Other Tools (Cursor, Codex CLI, Gemini CLI, Copilot, etc.)

Use [skills.sh](https://skills.sh/) to install skills into 40+ AI coding tools.

```bash
npx skills add smnbss/brain
```

Install specific skills or target specific tools:

```bash
npx skills add smnbss/brain --skill brain-pull-sources -a cursor -a codex
```

## Available Skills

### Brain (`brain`)

| Skill | Description |
|-------|-------------|
| `brain-pull-sources` | Export all external sources — ClickUp, Confluence, GDrive, Linear, GitHub, Medium, Metabase |
| `brain-rebuild-services` | Generate deep service `.AGENT.MD` docs from cloned GitHub repos |
| `brain-rebuild-memory` | Rebuild L2 (domain knowledge) and L1 (navigation MOCs) from `src/` + `outputs/services/` |
| `brain-git-sync` | Commit and push all brain changes |

### Commands

| Command | Description |
|---------|-------------|
| `/morning-start` | Update tools, sync brain, harvest meeting notes, prepare today's meetings |
| `/prepare-my-day` | Prepare agendas for today's deep dives and 1:1s |
| `/push-reports` | Push latest report outputs to ClickUp document pages |
| `/weekly-review` | Compile weekly summary from workflowy + x.com + linear outputs |

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
├── .claude-plugin/marketplace.json     # Registry of available plugins
├── plugins/
│   └── brain/
│       ├── .claude-plugin/plugin.json
│       ├── commands/
│       │   ├── morning-start.md
│       │   ├── prepare-my-day.md
│       │   ├── push-reports.md
│       │   └── weekly-review.md
│       └── skills/
│           ├── brain-pull-sources/
│           │   ├── SKILL.md
│           │   ├── bin/
│           │   ├── utils/
│           │   └── references/
│           ├── brain-git-sync/
│           │   └── SKILL.md
│           ├── brain-rebuild-memory/
│           │   └── SKILL.md
│           └── brain-rebuild-services/
│               └── SKILL.md
├── CLAUDE.md
├── AGENTS.md -> CLAUDE.md
└── README.md
```

**Design decisions:**
- Skills follow the [Agent Skills open standard](https://skills.sh/) (`SKILL.md` with YAML frontmatter) — compatible with 40+ tools
- Claude marketplace is the primary distribution (auto-updates); `npx skills` covers everything else
- One plugin (`brain`) groups all knowledge management skills together
