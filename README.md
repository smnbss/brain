# brain

[![skills.sh](https://img.shields.io/badge/skills.sh-compatible-blue)](https://skills.sh)

AI agent skills for building and syncing the WeRoad knowledge brain.

## Install

```bash
npx skills add smnbss/brain
```

## Skills

| Skill | Description |
|-------|-------------|
| `brain-pull-sources` | Export all external sources — ClickUp, Confluence, GDrive, Linear, GitHub, Medium, Metabase |
| `brain-rebuild-services` | Generate deep service `.AGENT.MD` docs from cloned GitHub repos |
| `brain-rebuild-memory` | Rebuild L2 (domain knowledge) and L1 (navigation MOCs) from `src/` + `outputs/services/` |
| `brain-git-sync` | Commit and push all brain changes |

## Required environment

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

## Quick start

1. **Populate sources** — invoke the `brain-pull-sources` skill to export all external data into `src/`
2. **Build service docs** — invoke the `brain-rebuild-services` skill to generate per-repo architecture docs
3. **Build memory** — invoke the `brain-rebuild-memory` skill to synthesize L2 and L1 navigation maps
4. **Save** — invoke the `brain-git-sync` skill to commit and push
