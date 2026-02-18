# Product Manager — Claude Code Plugin

GitHub issue management and session planning for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Keeps your backlog organized so you can focus on building.

## What it does

Two modes, triggered automatically based on context:

**Intake mode** — When you mention a bug, feature request, or any work item, the skill files a GitHub issue with the right labels and stops. No accidental implementation rabbit holes.

**Triage mode** — When you say `/pm` or ask "what should we work on?", the skill pulls all open issues, ranks them by priority/effort/impact, and recommends a session plan for your approval.

## Label scheme

Issues are labeled with priority (`P0`–`P3`) and type (`bug`, `feature`, `enhancement`, `chore`, `polish`, `perf`). The skill checks for existing labels first and reuses them where possible. You can override the scheme in your project's `CLAUDE.md`.

## Requirements

- [GitHub CLI (`gh`)](https://cli.github.com/) installed and authenticated
- A git repo with a GitHub remote

## Install

```
claude plugin add austinbrace/product-manager-skill
```

## Usage

The skill activates automatically when relevant. You can also invoke it directly:

- Report something: *"The login page throws a 500 when email is empty"* → files an issue
- Plan a session: `/pm` or *"What should we work on?"* → triages your backlog

## License

MIT
