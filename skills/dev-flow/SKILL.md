---
name: dev-flow
description: Standard workflow for feature work and bug fixes in any repo - stress-tested plan, then TDD implementation, then a review gate before the user commits. Use when starting a new feature or bug fix, or when the user types /dev-flow.
---

# Dev Flow

One session, one model. Do not switch models or start a new session unless the user says so.

Create a todo per phase and work them in order. Do not skip a phase because the task "looks small".

If the current project has its own overlay skill (see `project-overlay`), read that too: it overrides
this one wherever they disagree, and supplies the real commands for phase 3.

---

## Phase 1 - Plan

Route by task type. These are different problems with different entry points.

**Feature / new behaviour**
1. Ask the user to type `/grill-with-docs`. It is `disable-model-invocation: true`, so you
   cannot trigger it. Do not design anything before they have.
2. Then `superpowers:writing-plans`.

**Bug fix**
1. Diagnose FIRST: `superpowers:systematic-debugging` (or the `diagnosing-bugs` skill).
   Never design a fix for a bug you have not reproduced and explained.
2. Then `superpowers:writing-plans`.
3. Grill the fix only if it turns out to be non-trivial.

**Required output: a plan file on disk.**
A single long session will hit context compaction. Anything living only in the transcript is on
a timer. The plan file, plus any ADRs or `CONTEXT.md` updates, is what survives.

### STOP at the end of Phase 1

**Do not begin Phase 2 on your own. Ever.** When the plan file is written, stop and hand it to
the user: say the plan is ready, name the file, and wait.

The user decides when implementation starts, every time. This is not a formality to acknowledge
and move past. A plan being finished, obviously correct, small, or urgent is never a reason to
continue, and neither is the user having approved a similar plan before.

Resume only on an explicit instruction to start implementing. Silence is not consent, and
neither is "looks good" on the plan itself: approving a plan approves the plan, not the start of
the work. If it is ambiguous, ask.

---

## Phase 2 - Implement

**Default: `superpowers:subagent-driven-development`.** Do not implement inline.

Each plan task goes to its own subagent, which does TDD internally and returns a summary. The
subagent's context is discarded on return, so the main thread stays lean across a long session.
This is what makes one-session work survivable.

- **Pass `model: "opus"` explicitly** on each Agent call. Do not rely on inheritance: a default
  subagent model configured later would silently downgrade every task.
- Inside each subagent: `superpowers:test-driven-development` (or `tdd`). Failing test first,
  then implementation.
- **Batch the review checkpoints.** Let several tasks complete, then review once per coherent
  chunk. Do not gate every individual task.
- This phase authorises the Agent tool. Outside this workflow, do not spawn subagents unless
  asked.

Read the project's own rules (CLAUDE.md, AGENTS.md, docs/) and put them in the subagent brief.
Do not let a subagent pattern-match on neighbouring code: most codebases contain code predating
their current standards, so the file next door teaches the wrong thing.

Cost note: each subagent is a full Opus context. A plan with many tasks is correspondingly
expensive. That is the price of a lean main thread; say so up front rather than mid-run.

---

## Phase 3 - Review gate

Cheap correctness first, so no tokens are spent reviewing code that does not compile.

1. **Discover this project's gate.** Do not assume a command. Check `package.json` scripts,
   Makefile, CI config, or the project's CLAUDE.md. Look for a pre-push / check-all script.
2. **Fast checks** - unit tests and typecheck for the affected package only.
3. **Quality pass** - the BUILT-IN `/code-review`, `medium` by default. Escalate to `high` or
   `max` only for auth, data-layer, security, or cross-package changes.
4. **Architecture pass** - ask the user to type `/improve-codebase-architecture`
   (`disable-model-invocation: true`, you cannot trigger it). Worth it on sprawling or
   structural diffs, not on every task.
5. **Apply fixes.**
6. **Full gate** - run the project-wide check discovered in step 1.
7. **Stop.** Report what passed and what did not, with the actual output.

### Name collision: `code-review`

Two skills share this name and take incompatible arguments:

- **built-in `/code-review`** - effort `low|medium|high|xhigh|max`, `--fix`, `--comment`;
  reviews the current diff. This is the one step 3 means.
- **mattpocock `code-review`** - takes a commit/branch/tag to diff *since*, reviews Standards
  and Spec in parallel subagents, and does **not** accept effort levels. Use it deliberately
  for branch-vs-main reviews, never as a drop-in for step 3.

---

## Guardrails

- **Never start Phase 2 automatically.** Phase 1 ends with a plan handed over, then you wait for
  an explicit instruction to implement. The user starts the work, every time.
- **Never commit automatically.** Phase 3 ends with the work ready. The user commits.
- **Never create or switch git branches** without explicit instruction. Ask where work belongs.
- **Never `git add .`** - repos may track `.env*` files holding live keys. Stage by explicit path.
- Report failures faithfully, with output. Never claim completion on unverified work.
