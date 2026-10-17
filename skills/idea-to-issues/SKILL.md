---
name: idea-to-issues
description: Turn a developer's rough idea into well-scoped GitHub issues through a guided requirements conversation: choose-or-type questions, understanding checks until agent and developer are in sync, then slicing and issue creation. Issues get acceptance criteria, dependencies, and the queue label (default ready-for-agent) so issue-orchestration can pick them up. Use when the user wants to turn an idea, feature, or project into issues or tickets.
---

# Idea to Issues

Requirements gathering, then ticket writing. The developer owns the WHAT and WHY; you own the questions, the slicing, and the wording. Five steps: Ground, Guided interview, Sync check, Slice, Create. The goal is brain sync: before any issue exists, the developer has read your understanding and confirmed it matches theirs.

## Prerequisites

1. `gh auth status` passes and the target repo has Issues enabled (`gh repo view --json hasIssuesEnabled`). If disabled, say so and stop; never enable it yourself.
2. Target repo: the current repo, or the one the user names (`owner/name`). Pin it once and pass `--repo` to every `gh` call.
3. Queue label: the user's choice, else the `ready-for-agent` mapping in `docs/agents/triage-labels.md`, else `ready-for-agent`. Create it if missing (`gh label create`).

## 1. Ground

Before the first question, read what exists so you do not ask what the repo already answers: README, `CONTEXT.md`, `AGENTS.md` / `CLAUDE.md`, `docs/`, the stack (package manifests), and open issues (`gh issue list --state open --limit 50`) to avoid duplicates. New or empty repo: skip and say so.

## 2. Guided interview

Start by asking for the idea in their own words if they have not given it. Then guide with questions, one per message, each answerable in one click or one line:

- Every question offers 2 to 4 concrete options built from what you know so far (their words, the repo, the stack), plus a free-text "something else" path. Put your recommended option first and say why in a few words.
- Claude Code: use `AskUserQuestion` (it adds "Other" for typed input; use `multiSelect` when answers can combine). Other agents: a numbered list ending with "or type your own".
- Options must be real alternatives that change what gets built, not filler. If you cannot think of real options, ask an open question instead.
- After each answer, say in one line what it changed in your picture ("Got it: exports are per-invoice, not bulk.").

Cover this ground in order of what is most unclear (never read it out as a checklist):

- Problem and who has it; why now.
- What done looks like: observable outcomes, not implementation.
- Scope: goals, non-goals, and what would make it a failure even if it works.
- Constraints: stack, integrations, data, auth, deadlines, budget.
- Edge cases and failure behavior they care about.
- Priority: what ships first if only one thing could.

Stop when you could write acceptance criteria for every part without guessing. Usually 4 to 10 questions. "Just write them" from the user: skip to the sync check, with every unconfirmed point marked as an assumption.

## 3. Sync check

Show what you understood, so the developer can see whether you share the same picture. Keep it under 300 words:

```markdown
## My understanding
**Problem:** ...
**For:** ...
**Done when:** ...
**In scope:** ...
**Out of scope:** ...
**Constraints:** ...
**First slice:** ...

**I am confident about:** <points they stated>
**I am guessing:** <points you inferred; each one is a question>
```

Then ask with options: (1) In sync, slice it (2) Mostly, fix a few points (3) Not quite, let me re-explain. On 2 or 3, ask about the specific gaps (guided again), update the understanding, and show it again. Repeat until they pick "In sync" with nothing left under "I am guessing" that they care about. Never slice before that.

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

Show the full draft set in chat: titles, labels, dependency order, and each body. Ask with options: (1) Create all (2) Edit some first (3) Re-slice. Edits come back as changes to the draft, then ask again. This is the only approval step; never create issues before it.

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
