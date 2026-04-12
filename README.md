# brain

AI agent commands and skills for building and syncing the WeRoad knowledge brain.

## What's inside

### Commands (`.agents/commands/`)

| Command | Purpose |
|---------|---------|
| `brain-sync.md` | Orchestrate a full brain sync — pull external sources, update memory layers, write digest |
| `rebuild-memory.md` | Rebuild L2 (domain knowledge) and L1 (navigation MOCs) from `src/` + `outputs/services/` |
| `rebuild-services-docs.md` | Generate deep service `.AGENT.MD` docs from cloned GitHub repos |
| `git-sync.md` | Commit and push all brain changes |
| `morning-start.md` | Daily routine — tool updates, brain sync, meeting harvest, day prep |
| `dbt-build.md` | Run dbt builds (optional) |

### Skills (`.agents/skills/`)

| Skill | Purpose |
|-------|---------|
| `weroad-brain-sync/` | The core sync engine — exports ClickUp, Confluence, GDrive, Linear, GitHub, Medium, Metabase |

## Install

### Option A: `super install`

Add to your `super.config.yaml`:

```yaml
skills:
  weroad-brain:
    source: smnbss/brain
    description: WeRoad brain sync commands and skills
    enabled: true
```

Then run:

```bash
super install
```

After install, copy the commands into your brain project:

```bash
cp -r .agents/skills/weroad-brain/.agents/commands/* /path/to/your/brain/.agents/commands/
cp -r .agents/skills/weroad-brain/.agents/skills/* /path/to/your/brain/.agents/skills/
```

### Option B: Direct clone

```bash
git clone https://github.com/smnbss/brain.git /tmp/brain
cp -r /tmp/brain/.agents/commands/* /path/to/your/brain/.agents/commands/
cp -r /tmp/brain/.agents/skills/* /path/to/your/brain/.agents/skills/
```

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

1. **Populate sources** — run `/brain-sync` to export all external data into `src/`
2. **Build service docs** — run `/rebuild-services-docs` to generate per-repo architecture docs
3. **Build memory** — run `/rebuild-memory` to synthesize L2 and L1 navigation maps
4. **Save** — run `/git-sync` to commit and push
