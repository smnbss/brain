---
name: brain-linear-process-tasks
description: >
  Pick the next task issue from a Linear project, gather context, implement it,
  and mark it done. Use when the user says "process task", "implement next task",
  "work on next issue", or passes a Linear project URL and asks to implement tasks.
---

# Linear — Process Tasks

Pick the first issue in Todo from a Linear project, gather full context, ask
clarifying questions, implement the change, run tests, commit, and mark done.

## Input

The user can provide a Linear project URL, e.g.:
```
https://linear.app/weroad/project/super-edc40903e247/issues
```

Optionally, the user can also provide a specific issue URL to work on:
```
https://linear.app/weroad/issue/SIM-7
```

**If no project URL is provided**, read the `LINEAR_LOOP` env var from `.env.local` in the
project root. It contains a semicolon-separated list of Linear project URLs:
```bash
# Example: LINEAR_LOOP=https://linear.app/weroad/project/super-xxx/overview;https://linear.app/weroad/project/others-yyy/issues;
```

Process **each project** in the list sequentially — find the first eligible task in
Todo for each project and implement it before moving to the next.

If neither a URL nor `LINEAR_LOOP` is available, **stop immediately** and ask:
> "I need a Linear project URL to pick tasks from. Example: `https://linear.app/weroad/project/<slug>/issues`"

## Step 1 — Parse and fetch

Extract the project slug from the URL (segment after `/project/`, before next `/`).

If a specific issue URL was provided, extract the issue identifier (e.g., `SIM-7`).

Fetch project details and issues in parallel:
1. `get_project` with `query: "<slug>"`
2. If a specific issue was given: `get_issue` with `query: "<identifier>"`
3. If no specific issue: `list_issues` with `project: "<slug>"`, `state: "Todo"`, `limit: 100`

## Step 2 — Find the target issue

**If a specific issue was provided:** Use that issue directly.

**If picking from Todo:** From the returned issues, find the **first** issue
(sorted by `createdAt` ascending — oldest first, or by priority if available).

Skip issues whose title starts with `"Idea:"` (case-insensitive) — those are
handled by `brain-linear-process-idea`.

**If no eligible issue is found in Todo:**
Stop and report:
> "No task issues found in Todo for project <name>. Nothing to implement."

If there are issues in other statuses (Backlog, In Progress, In Review), mention
them so the user knows what exists.

## Step 3 — Read the full context

Fetch the complete issue using `get_issue` with the issue identifier.

Also read:
- The **project description** (from Step 1) — this provides the broader project context.
- Any **comments** on the issue using `list_comments` with `issueId: "<ID>"`.
- Any **sub-issues** if referenced.

Gather all of this into your working context.

## Step 4 — Ask before proceeding

**Before writing any code**, present the user with:

1. **Issue summary:** ID, title, description (condensed).
2. **Your understanding:** What needs to be done, in your own words.
3. **Approach:** How you plan to implement it (files to change, strategy).
4. **Questions:** Anything unclear or ambiguous — ask now.

Wait for user confirmation before proceeding. If the user provides clarifications,
incorporate them.

## Step 5 — Move to In Review

Use the `save_issue` Linear MCP tool to transition the issue:
- `id`: the issue identifier (e.g., "SIM-7")
- `state`: "In Review"

Report: `"Moved <ID>: <title> to In Review — starting implementation"`

## Step 6 — Implement

Implement the changes following best practices:

1. **Read before writing** — understand the existing code before modifying it.
2. **Make focused changes** — only change what the issue requires.
3. **Follow existing patterns** — match the codebase's conventions.
4. **Run tests** — execute the project's test suite and ensure tests pass.
5. **Fix issues** — if tests fail, fix them before proceeding.

## Step 7 — Commit

Once tests pass, commit the changes to git:

1. Stage only the relevant files (not unrelated changes).
2. Write a commit message referencing the issue:
   ```
   feat: <short description> [<ID>]
   ```
3. Do NOT push unless the user explicitly asks.

## Step 8 — Update the project

Use the `save_status_update` Linear MCP tool to publish a project update:
- `type`: `"project"`
- `project`: the project ID or slug from Step 1
- `health`: choose based on the implementation:
  - `onTrack` if the task was completed cleanly
  - `atRisk` if the task was completed but with caveats or partial implementation
  - `offTrack` if the task could not be completed
- `body`: a concise Markdown summary including:
  - The issue ID and title
  - What was implemented
  - Any caveats or follow-ups needed

## Step 9 — Mark as Done

Use `save_issue` to transition the issue:
- `id`: the issue identifier
- `state`: "Done"

Add a comment on the issue summarizing what was done:
> "Implemented and committed. <one-line summary of what changed>.
> Commit: `<short hash>` — tests passing."

## Step 10 — Report

Print:
- Issue ID, title, and URL
- What was implemented (2-3 sentences)
- Commit hash
- Test results (pass/fail)
- Confirmation that the issue was marked Done
- Confirmation that a project status update was posted
- Any follow-ups or caveats
