# Brain — AI Skills Repository

AI agent skills for building and syncing the WeRoad knowledge brain. Skills follow the [Agent Skills open standard](https://skills.sh/) (`SKILL.md` with YAML frontmatter) and are compatible with 40+ tools including Claude Code, Gemini CLI, Codex CLI, Cursor, Copilot, and more.

**Distribution:** `npx skills add smnbss/brain` — one command, all CLIs.

## Repository Structure

```
skills/<skill-name>/
  SKILL.md                    # The skill definition
  bin/                        # Optional: shell scripts
  utils/                      # Optional: Python utilities
  references/                 # Optional: reference templates
```

## Skill Format

Every `SKILL.md` has YAML frontmatter and a markdown body:

```markdown
---
name: skill-name
description: When to use this skill (one sentence — this is shown to the AI as context for activation)
---

# Skill Title

## When to Use
...

## Instructions
...
```

**Frontmatter fields:**
- `name` — kebab-case, matches the folder name
- `description` — tells the AI *when* to activate this skill (be specific)

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

## Naming Conventions

- **Skill folders**: kebab-case (`brain-pull-sources`, `brain-rebuild-memory`)
- **Skill files**: always `SKILL.md` (uppercase)
- **Reference files**: kebab-case `.md` files in `references/`
- **Shell scripts**: kebab-case in `bin/`

## Commits

Use [Conventional Commits](https://www.conventionalcommits.org/) for all commit messages:

```
<type>: <short description>
```

Common types: `feat`, `fix`, `docs`, `chore`, `refactor`

## Rules

- **No secrets** — never include API keys, passwords, or tokens in skills
- **Actionable instructions** — every skill must change how the AI operates
- **One concern per skill** — each skill covers one pipeline phase
