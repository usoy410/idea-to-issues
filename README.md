# idea-to-issues

Agent skill that turns a developer's rough idea into well-scoped GitHub issues through a short requirements conversation. The output is ready for [software-engineering-orchestration](https://github.com/usoy410/software-engineering-orchestration): issues carry acceptance criteria, dependency order, and the `ready-for-agent` queue label.

## Prerequisites

- A GitHub repository with Issues enabled (Settings > Features > Issues).
- `gh` installed and authenticated (`gh auth status`) with permission to create issues and labels.

## Install

Per repository:

```bash
npx skills@latest add usoy410/idea-to-issues -a claude-code
```

Global:

```bash
npx skills@latest add usoy410/idea-to-issues -a claude-code -a antigravity -g
```

Manual: copy `skills/*` into `.claude/skills/` (repo) or `~/.claude/skills/` (global).

## Use

> /idea-to-issues I want members to be able to export their invoices as PDF

## How it works

```text
idea
  |
  v
ground     read README, CONTEXT.md, docs, stack, open issues
  |
  v
interview  one question at a time: problem, done, scope, constraints, edge cases, priority
  |
  v
play back  short brief; repeat until the developer agrees
  |
  v
slice      vertical slices, one PR each, dependency ordered
  |
  v
confirm    full draft shown once; nothing is created before "yes"
  |
  v
create     gh issue create in order; queue label on unblocked issues; tracking issue if > 3
  |
  v
/issue-orchestration picks them up
```

Each issue has: Context, Behavior, Acceptance criteria, Out of scope, Notes, `Depends on`.

## Customize

Queue label: say "use label X", or map it in `docs/agents/triage-labels.md`.
