---
name: brain-linear-process-idea
description: >
  Pick the next "Idea:" issue from a Linear project, move it to In Progress, and
  iteratively improve it across 3 passes. Use when the user says "process idea",
  "work on next idea", "improve ideas", or passes a Linear project URL and asks
  to process or refine ideas.
---

# Linear — Process Idea

Pick the first "Idea:" issue in Todo from a Linear project, move it to In Progress,
and iteratively refine it across 3 structured passes. Each pass deepens the analysis
and is tracked with a counter in the issue description.

## Input

The user can provide a Linear project URL, e.g.:
```
https://linear.app/weroad/project/super-edc40903e247/issues
```

**If no URL is provided**, read the `LINEAR_LOOP` env var from `.env.local` in the
project root. It contains a semicolon-separated list of Linear project URLs:
```bash
# Example: LINEAR_LOOP=https://linear.app/weroad/project/super-xxx/overview;https://linear.app/weroad/project/others-yyy/issues;
```

Process **each project** in the list sequentially — find the first "Idea:" issue in
Todo for each project and process it before moving to the next.

If neither a URL nor `LINEAR_LOOP` is available, **stop immediately** and ask:
> "I need a Linear project URL to process ideas from. Example: `https://linear.app/weroad/project/<slug>/issues`"

## Step 1 — Parse and fetch

Extract the project slug from the URL (segment after `/project/`, before next `/`).

Fetch project details and issues in parallel:
1. `get_project` with `query: "<slug>"`
2. `list_issues` with `project: "<slug>"`, `state: "Todo"`, `limit: 100`

## Step 2 — Find the target issue

From the returned issues, find the **first** issue whose title starts with `"Idea:"` 
(case-insensitive). Sort by `createdAt` ascending (oldest first) to process ideas in
the order they were created.

**If no "Idea:" issue is found in Todo:**
Stop and report:
> "No 'Idea:' issues found in Todo for project <name>. Nothing to process."

If there are "Idea:" issues in other statuses (Backlog, In Progress), mention them
so the user knows what exists.

## Step 3 — Move to In Progress

Use the `save_issue` Linear MCP tool to transition the issue:
- `id`: the issue identifier (e.g., "SIM-3")
- `state`: "In Progress"

Report: `"Moved <ID>: <title> to In Progress"`

## Step 4 — Read the full context

Fetch the complete issue using `get_issue` with the issue identifier.

Also read:
- The **project description** (from Step 1) — this provides the broader project context.
- Any **comments** on the issue using `list_comments` with `issueId: "<ID>"`.

Gather all of this into your working context.

## Step 5 — Three improvement passes

Perform exactly 3 passes on the idea. After each pass, update the issue description
using `save_issue`. Each pass builds on the previous one.

### Pass counter

Maintain a progress tracker at the TOP of the issue description:

```markdown
<!-- idea-processing: pass N/3 | last-updated: YYYY-MM-DD HH:MM -->
```

If this marker already exists (from a previous run), read the current pass number
and continue from where it left off. If all 3 passes are already done, report that
and ask the user if they want to reset and do 3 more.

### Pass 1 — Research & Expand

**Goal:** Understand the idea deeply and expand it with research.

- What problem does this idea solve? Who benefits?
- What are the key assumptions? Are they valid?
- Research: use web search, DeepWiki, or brain knowledge (`qmd query`) to find
  relevant prior art, similar implementations, documentation, or competing approaches.
- Expand the description with findings: background, prior art, relevant links.
- List 3-5 concrete approaches or options to implement this idea.

**Update the issue description** with the expanded context. Preserve the original
idea text — add to it, don't replace it.

Format:
```markdown
<!-- idea-processing: pass 1/3 | last-updated: YYYY-MM-DD HH:MM -->

## Original Idea

<original description, preserved verbatim>

## Pass 1 — Research & Expansion

### Problem Statement
<crisp problem statement>

### Background & Prior Art
<research findings, links, similar approaches>

### Options
1. **<Option name>** — <description, pros, cons>
2. **<Option name>** — <description, pros, cons>
3. **<Option name>** — <description, pros, cons>
```

### Pass 2 — Evaluate & Recommend

**Goal:** Critically evaluate each option and recommend a path forward.

- For each option from Pass 1, assess:
  - **Feasibility:** How hard is it? What's the effort (T-shirt size)?
  - **Impact:** How much does it move the needle on the problem?
  - **Risk:** What could go wrong? Dependencies? Blockers?
  - **Alignment:** Does it fit the project goals (from project description)?
- Score each option on a simple matrix (feasibility x impact).
- Recommend a preferred option with clear reasoning.
- Identify what you'd need to validate before committing.

**Append** to the issue description:
```markdown
## Pass 2 — Evaluation & Recommendation

### Option Comparison

| Option | Feasibility | Impact | Risk | Score |
|--------|-------------|--------|------|-------|
| ...    | ...         | ...    | ...  | ...   |

### Recommendation
<which option and why>

### Open Questions / Validation Needed
- <question 1>
- <question 2>
```

Update the pass counter to `pass 2/3`.

### Pass 3 — Action Plan

**Goal:** Turn the recommendation into concrete next steps.

- Break the recommended option into actionable tasks (3-7 tasks).
- For each task: what needs to happen, rough effort, any dependencies.
- Identify the first concrete step someone could take TODAY.
- Flag any decisions that need human input before proceeding.
- Write a one-paragraph "elevator pitch" summary of the refined idea.

**Append** to the issue description:
```markdown
## Pass 3 — Action Plan

### Summary
<one paragraph elevator pitch of the refined idea>

### Tasks
- [ ] <task 1> — <effort estimate>
- [ ] <task 2> — <effort estimate>
- [ ] <task 3> — <effort estimate>

### First Step
<what to do right now>

### Decisions Needed
- <decision 1 — who needs to decide>
```

Update the pass counter to `pass 3/3`.

## Step 6 — Final update

After all 3 passes, leave the issue in "In Progress" (the user decides when to move
it to Done or back to Todo).

Add a comment on the issue summarizing what was done:
> "Completed 3-pass idea refinement. The description now contains: research & options
> (Pass 1), evaluation & recommendation (Pass 2), and an action plan (Pass 3).
> Ready for human review."

## Step 7 — Create a project status update

Use the `save_status_update` Linear MCP tool to publish a project update:
- `type`: `"project"`
- `project`: the project ID or slug from Step 1
- `health`: choose based on the refined idea:
  - `onTrack` if the idea has a clear path forward and actionable next steps
  - `atRisk` if significant open questions or blockers remain
  - `offTrack` if the idea is not viable or requires major rethinking
- `body`: a concise Markdown summary of the idea refinement. Include:
  - The issue ID and title that was processed
  - Which passes were completed
  - The recommended option
  - The first actionable step
  - Any decisions needed

Example body:
```markdown
Refined SIM-3: Automate weekly report generation.

**Progress:**
- Completed 3-pass idea refinement (Pass 1: research & options, Pass 2: evaluation, Pass 3: action plan)
- **Recommended option:** Build a Python script triggered by GitHub Actions
- **First step:** Draft the script to pull data from the analytics API

**Decisions needed:** Confirm budget for GitHub Actions runners.
```

## Step 8 — Report

Print:
- Issue ID, title, and URL
- Which passes were completed (1/3, 2/3, 3/3)
- The recommended option (one sentence)
- The first actionable step
- Confirmation that a project status update was posted
- A reminder that the issue is in "In Progress" awaiting human review
