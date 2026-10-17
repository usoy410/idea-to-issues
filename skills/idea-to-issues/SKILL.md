---
name: idea-to-issues
description: Turn a developer's rough idea into well-scoped GitHub issues through a short requirements conversation (interview, play back, slice, confirm, create). Issues get acceptance criteria, dependencies, and the queue label (default ready-for-agent) so issue-orchestration can pick them up. Use when the user wants to turn an idea, feature, or project into issues or tickets.
---

# Idea to Issues

Requirements gathering, then ticket writing. The developer owns the WHAT and WHY; you own the questions, the slicing, and the wording. Five steps: Ground, Interview, Play back, Slice, Create.

## Prerequisites

1. `gh auth status` passes and the target repo has Issues enabled (`gh repo view --json hasIssuesEnabled`). If disabled, say so and stop; never enable it yourself.
2. Target repo: the current repo, or the one the user names (`owner/name`). Pin it once and pass `--repo` to every `gh` call.
3. Queue label: the user's choice, else the `ready-for-agent` mapping in `docs/agents/triage-labels.md`, else `ready-for-agent`. Create it if missing (`gh label create`).

## 1. Ground

Before the first question, read what exists so you do not ask what the repo already answers: README, `CONTEXT.md`, `AGENTS.md` / `CLAUDE.md`, `docs/`, the stack (package manifests), and open issues (`gh issue list --state open --limit 50`) to avoid duplicates. New or empty repo: skip and say so.

## 2. Interview

A conversation, not a form. One question per message, plain language, anchored in specifics ("walk me through the last time you needed this"). Get them describing before offering options; offer a short menu only when they are stuck, and propose something concrete when they say "you decide".

Cover this ground (never read it out as a checklist):

- Problem and who has it; why now.
- What done looks like: observable outcomes, not implementation.
- Scope: goals, non-goals, and what would make it a failure even if it works.
- Constraints: stack, integrations, data, auth, deadlines, budget.
- Edge cases and failure behavior they care about.
- Priority: what ships first if only one thing could.

Stop interviewing when you could write acceptance criteria for every part without guessing. Usually 4 to 10 questions. "Just write them" from the user: stop now, and mark every unconfirmed assumption in the issues.

## 3. Play back

Write a short brief (under 300 words) in the chat: problem, users, outcomes, scope in/out, constraints, open questions. Mark your guesses as guesses. Ask what is wrong or missing. Repeat until they agree. Do not slice before agreement.

## 4. Slice

Split into vertical slices: each issue delivers one end-to-end, testable behavior (UI or API through storage), small enough for one PR. No "set up the database" or "write tests" issues unless the slice is genuinely infrastructure. Order by dependency; the first issue is the thinnest working path.

Each issue uses this body:

```markdown
## Context
<why this exists, 1-3 sentences, link the parent brief/epic>

## Behavior
<what a user or caller can do when this is done>

## Acceptance criteria
- [ ] <observable, testable statement>
- [ ] ...

## Out of scope
- <explicitly excluded>

## Notes
<constraints, relevant files, edge cases, assumptions marked "Assumption:">

Depends on: #<n> | none
```

Title: imperative, specific, under 70 chars ("Let members reset password via email link"). Labels: `enhancement` or `bug`, plus any area labels the repo already uses. Do not invent new area labels.

More than 3 issues: also draft one tracking issue (the brief plus a task list of the slices).

## 5. Confirm and create

Show the full draft set in chat: titles, labels, dependency order, and each body. Ask once: create these? Edits come back as changes to the draft, then ask again. This is the only approval step; never create issues before it.

On approval, create in dependency order so `Depends on` can use real numbers:

```sh
gh issue create --repo <repo> --title "<title>" --body-file <file> --label "<labels>"
```

The queue label goes only on issues with no unresolved `Depends on` and no `Assumption:` the user flagged as unsure; the rest get created without it and the summary says why. Edit the tracking issue last with the real numbers.

## Rules

- Never create, edit, close, or relabel issues outside this run's draft.
- Never put secrets, credentials, or personal data in an issue body.
- Unclear after the interview: put it under "open questions" in the tracking issue, not in acceptance criteria.

## Report

End with: repo, created issues (number, title, URL, queued or not), tracking issue, and open questions. Suggest `/issue-orchestration process the <queue-label> issues` when any are queued.
