# Product Manager — Claude Code Plugin

GitHub issue management and session planning for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Keeps your backlog organized so you can focus on building.

## What it does

Two modes, triggered automatically based on context:

**Intake mode** — When you mention a bug, feature request, or any work item, the skill files a GitHub issue with the right labels and stops. No accidental implementation rabbit holes.

**Triage mode** — When you say `/pm` or ask "what should we work on?", the skill:

1. Spawns a subagent to research all open issues (keeps your main context clean)
2. Ranks issues by project goal alignment, priority, dependencies, effort, and impact
3. Presents a session plan for your approval
4. Recommends an execution strategy (parallel, sequential, plan-first, or mixed)
5. Waits for your go-ahead before touching anything

## Features

- **Context caching** — On first run, discovers which project docs are useful (CLAUDE.md, README, SPEC.md) and caches the result. Subsequent invocations skip the search.
- **Respects existing labels** — Checks `gh label list` before creating anything. Reuses labels that map to the same concept. Override the scheme in your project's CLAUDE.md.
- **Goal-aligned triage** — Priorities aren't just P0 > P1. The skill reads your project docs and ranks issues by what advances your stated goals. Flags stale labels that conflict with current milestones.
- **Cross-tool compatible** — Works with both Claude Code (`.claude/`) and Kiro (`.kiro/`).

## Label scheme

Priority: `P0` (critical/blocking), `P1` (high/soon), `P2` (medium/nice-to-have), `P3` (low/backlog)

Type: `bug`, `feature`, `enhancement`, `chore`, `polish`, `perf`

## Requirements

- [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated
- A git repo with a GitHub remote

## Install

In Claude Code, run:

```
/plugin marketplace add atbrace/product-manager-skill
```

Then enable the `product-manager` plugin when prompted.

## Usage

The skill activates automatically when relevant. You can also invoke it directly:

- Report something: *"The login page throws a 500 when email is empty"* → files an issue
- Plan a session: `/pm` or *"What should we work on?"* → triages your backlog

## License

MIT
