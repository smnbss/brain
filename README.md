# brain

AI agent skills for building and syncing the WeRoad knowledge brain.

## What's inside

### Skills (`.agents/skills/`)

| Skill | Purpose |
|-------|---------|
| `brain-brain-sync/` | Orchestrate a full brain sync — pull external sources, update memory layers, write digest |
| `brain-git-sync/` | Commit and push all brain changes |
| `brain-rebuild-memory/` | Rebuild L2 (domain knowledge) and L1 (navigation MOCs) from `src/` + `outputs/services/` |
| `brain-rebuild-git/` | Generate deep service `.AGENT.MD` docs from cloned GitHub repos |
| `brain-pull-sources/` | The core sync engine — exports ClickUp, Confluence, GDrive, Linear, GitHub, Medium, Metabase |

## Install

### Option A: `super install`

Add to your `super.config.yaml`:

```yaml
skills:
  weroad-brain:
    source: smnbss/brain
    description: WeRoad brain sync skills
    enabled: true
```

Then run:

```bash
super install
```

After install, copy the skills into your brain project:

```bash
cp -r .agents/skills/weroad-brain/.agents/skills/* /path/to/your/brain/.agents/skills/
```

### Option B: Direct clone

```bash
git clone https://github.com/smnbss/brain.git /tmp/brain
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

1. **Populate sources** — invoke the `brain-brain-sync` skill to export all external data into `src/`
2. **Build service docs** — invoke the `brain-rebuild-git` skill to generate per-repo architecture docs
3. **Build memory** — invoke the `brain-rebuild-memory` skill to synthesize L2 and L1 navigation maps
4. **Save** — invoke the `brain-git-sync` skill to commit and push
