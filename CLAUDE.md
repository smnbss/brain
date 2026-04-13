# Brain — AI Skills Repository

AI agent skills for building and syncing the WeRoad knowledge brain. Skills follow the Agent Skills open standard (`SKILL.md` with YAML frontmatter) and are compatible with 40+ tools including Claude Code, Cursor, Codex CLI, Gemini CLI, GitHub Copilot, and more.

**Distribution:** Claude Code users get skills via the plugin marketplace (auto-updates). All other tools use `npx skills add`.

## Repository Structure

```
plugins/brain/
  .claude-plugin/plugin.json    # Plugin metadata
  commands/<command-name>.md    # Slash commands (invoked via /command-name)
  skills/<skill-name>/
    SKILL.md                    # The skill definition
    bin/                        # Optional: shell scripts
    utils/                      # Optional: Python utilities
    references/                 # Optional: reference templates
```

The marketplace registry is at `.claude-plugin/marketplace.json`.

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

## Available Commands

| Command | Description |
|---------|-------------|
| `/morning-start` | Update tools, sync brain, harvest meeting notes, prepare today's meetings |
| `/prepare-my-day` | Prepare agendas for today's deep dives and 1:1s |
| `/push-reports` | Push latest report outputs to ClickUp document pages |
| `/weekly-review` | Compile weekly summary from workflowy + x.com + linear outputs |

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
