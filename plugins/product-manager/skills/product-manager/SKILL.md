---
name: product-manager
description: >
  Use when the user reports a bug, feature request, enhancement, issue,
  roadmap item, or anything that should be tracked. Also use when the user
  asks to plan a session, triage issues, decide what to work on, asks
  "what should we work on", or invokes /pm. Use when the user describes
  broken behavior, wants new functionality, or mentions backlog items.
---

# Product Manager

GitHub issue management and session planning. Two modes: intake (file issues) and triage (plan sessions).

## Mode Detection

If the user's message describes a specific bug, feature, enhancement, or roadmap item: **intake mode**.
If the user invoked /pm, asked what to work on, or wants to plan the session: **triage mode**.

## Label Scheme

These are the default labels. To customize, check for a project-level CLAUDE.md with a label scheme override. If one exists, use that instead.

**Priority:** P0 (critical/blocking), P1 (high/soon), P2 (medium/nice-to-have), P3 (low/backlog)

**Type:** bug, feature, enhancement, chore, polish, perf

Before creating labels, run `gh label list` to check what already exists. Reuse existing labels where they map to the same concept (e.g., if the repo already has `severity/critical`, use that instead of creating `P0`). Only create missing labels with `gh label create`:
- P0: color B60205, P1: color D93F0B, P2: color FBCA04, P3: color 0E8A16
- bug: color D73A4A, feature: color 0075CA, enhancement: color A2EEEF
- chore: color CFD3D7, polish: color D4C5F9, perf: color F9D0C4

## Repo Detection

Detect the repo from `git remote get-url origin`. Parse owner/repo for `gh` commands.
If not a git repo or no GitHub remote, tell the user and stop.

## Project Context Discovery

Project context helps assess priority and write better issues. Use this lookup strategy to find it efficiently.

**Step 0: Determine config directory.** Check which tool you're running in. Use the first directory that exists: `.claude/`, `.kiro/`, or fall back to `.claude/`. The cache file is `<config-dir>/product-manager.local.md`.

**Step 1: Check cache.** Read the cache file. If it exists and has a `project_docs` field in its YAML frontmatter, read only those files. Skip to done.

**Step 2: First-run discovery.** If the cache file doesn't exist, check these files in order (root of repo only — do not recurse):
1. `CLAUDE.md` or `KIRO.md` — whichever matches the tool. Already in context, no read needed. Use if it has a codebase overview.
2. `README.md` — read only the first 100 lines. Use if it describes what the project does.
3. `SPEC.md` — read if it exists.

Stop as soon as you have enough context to understand what the project does and assess severity. Do not search further.

**Step 3: Cache the result.** Write the cache file with the paths that were useful:

```yaml
---
project_docs:
  - CLAUDE.md
  - README.md
---
```

On subsequent invocations, the cache is read in Step 1 and no discovery is needed.

---

## INTAKE MODE

<HARD-GATE>
Intake mode ALWAYS stops after creating the issue. NEVER start implementation.
Do not suggest fixing it. Do not "take a quick look". Do not pivot to coding.
File the issue and stop. The ONLY output after issue creation is the issue URL.
</HARD-GATE>

When the user reports a problem, feature idea, or any work item:

**Step 1: Parse the report.** Extract the core issue. Identify the type:
- bug: something is broken or behaves incorrectly
- feature: entirely new capability that doesn't exist yet
- enhancement: improving something that already works
- chore: maintenance, tooling, infrastructure
- polish: visual or UX refinement
- perf: performance optimization

**Step 2: Read project context.** Follow the Project Context Discovery steps above. Use that context to assess severity. Assign a priority:
- P0: Blocks all progress or causes data loss
- P1: Significantly impacts core functionality, should fix soon
- P2: Noticeable but workaround exists, or non-core feature
- P3: Minor inconvenience, cosmetic, or nice-to-have

**Step 3: Check for duplicates.** Run `gh issue list --state open --json number,title,labels` and scan for similar issues. If a likely duplicate exists, show it to the user and ask: update the existing issue, or create a new one?

**Step 4: Draft the issue.** Present to the user for approval before creating:

```
Title: [concise, imperative description]
Labels: [type], [priority]
Body:
  ## Description
  [What is the issue]

  ## Steps to Reproduce (for bugs)
  1. ...
  2. ...

  ## Expected Behavior
  [What should happen]

  ## Actual Behavior
  [What happens instead]

  ## Acceptance Criteria (for features/enhancements)
  - [ ] ...
```

**Step 5: Create the issue.** After user approves, run:
`gh issue create --title "..." --body "..." --label "type,priority"`

**Step 6: Report the URL and stop.**

### Intake Red Flags

These thoughts mean STOP - you are about to violate the intake boundary:

| Thought | Correction |
|---------|------------|
| "Let me also fix this quickly" | No. File the issue and stop. |
| "This is simple, I'll just do it" | File it. Triage decides when to work on it. |
| "I don't need an issue for this" | ALL work is tracked. No exceptions. |
| "Let me take a quick look at the code" | Only look to assess priority. Do not start fixing. |
| "While I'm here, I might as well..." | You are not here to fix. You are here to file. |

---

## TRIAGE MODE

When the user invokes /pm or asks what to work on:

**Step 1: Gather context.**
- Run `gh issue list --state open --json number,title,labels,body,createdAt` to get all open issues
- Follow the Project Context Discovery steps above for domain context
- Check recent git log for what was worked on last

**Step 2: Assess and rank.** For each open issue, evaluate:
- **Alignment with project goals** — Use the project context (CLAUDE.md, README, SPEC) to understand what the project is trying to achieve. Issues that advance the project's stated goals or current milestone rank higher, regardless of label.
- **Priority label** (P0 > P1 > P2 > P3). If unlabeled, assign one now. If a label conflicts with project goals (e.g., a P3 issue is central to the current milestone), flag the discrepancy and recommend re-labeling.
- **Dependencies** between issues (does fixing X unblock Y?)
- **Effort estimate** (small: <30min, medium: 30min-2hr, large: >2hr). You may glance at relevant source files to calibrate effort, but do not start fixing anything.
- **Impact** on user experience

**Step 3: Recommend a session plan.** Present in this format:

```
Recommended session plan:
1. #N - Title (priority, type, effort)
   Reason: ...
2. #N - Title (priority, type, effort)
   Reason: ...

Deferring to a future session:
- #N - Title (priority, type, effort)
  Reason: ...
```

Group related issues that can be fixed together. Lead with the highest-impact work.

**Step 4: Ask for approval.** Wait for the user to approve, reorder, or adjust the plan. Do NOT start working until the user confirms.

**Step 5: Transition to implementation.** Once approved, begin implementation planning for the first issue in the plan. For non-trivial work, use EnterPlanMode to design the approach before coding. For small fixes, proceed directly. Always reference the GitHub issue number in commits.

### Triage Red Flags

| Thought | Correction |
|---------|------------|
| "I'll just start on the obvious one" | Present the full plan first. Get approval. |
| "No need to check issues, user told me what to do" | Always check GitHub for full context. |
| "I'll skip the low-priority ones in the list" | Show ALL open issues. User decides what to defer. |
| "This issue doesn't need an effort estimate" | Every issue gets priority + effort + impact assessment. |
| "The most recent commit tells me what to prioritize" | Git recency shows momentum, not priority. Use project goals. |
| "Labels already capture the right priority" | Labels can go stale. Validate against current project goals. |
