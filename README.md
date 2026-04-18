# brain — ARCHIVED

> **This repository has been archived.** The skills have moved to [smnbss/super](https://github.com/smnbss/super).

## Why

Brain skills depended on a sample config (`brain.config.yml`) that had to be installed somewhere. Super is the obvious install point — it already manages `~/.super/` — so merging brain into super lets a single `install.sh` deliver the skills, the sample config, and `qmd` in one shot.

## Migration

```bash
# Install super (now includes all brain-* skills and the config sample)
curl -fsSL https://raw.githubusercontent.com/smnbss/super/main/install.sh | bash
```

After install:

- Skills live in `~/.super/skills/brain-*/`
- Sample config is copied to `~/.super/brain.config.yml` (edit to match your org)
- `qmd` is installed globally via npm (for `brain-reindex` + `qmd query`)

## Skills moved

All 14 brain skills moved to `smnbss/super/skills/`:

`brain-pull-sources`, `brain-rebuild-services`, `brain-rebuild-memory`, `brain-reindex`, `brain-git-sync`, `brain-pull-my-meeting-notes`, `brain-prepare-my-one-on-one`, `brain-prepare-my-deep-dives`, `brain-morning-start`, `brain-push-reports`, `brain-weekly-review`, `brain-linear-create-project-context`, `brain-linear-process-ideas`, `brain-linear-process-tasks`.
