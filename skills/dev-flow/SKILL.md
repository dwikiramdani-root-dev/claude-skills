---
name: dev-flow
description: Standard workflow for feature work and bug fixes in any repo - stress-tested plan, then TDD implementation, then a review gate and a draft PR. Use when starting a new feature or bug fix, or when the user types /dev-flow.
---

# Dev Flow

One session, one model. Do not switch models or start a new session unless the user says so.

Create a todo per phase and work them in order. Do not skip a phase because the task "looks small".

**Read the project's overlay skill before Phase 0** (see `project-overlay`). It overrides this file
wherever they disagree, and it supplies what this file deliberately does not know: the real gate
commands, the branch naming convention, the ticket system, and whether this project uses PRs at
all. If the repo has no overlay, say so once - from there on you are inferring conventions that
should have been written down.

---

## Phase 0 - Ticket intake

Ask for all of this in one message, before anything else:

1. **Which input are you giving me** - the full ticket pasted in, or your own description and
   instructions? Ask; do not choose. A pasted ticket is not automatically the whole story, and a
   user's summary is not automatically complete.
2. **Ticket number and link.** Both. The number names the branch, the link goes in the PR body.
   Asking now means never asking twice.

No ticket for this work? Say so and continue, but ask what the branch should be called - the
naming convention has lost its input.

---

## Phase 1 - Plan

Route by task type. These are different problems with different entry points.

**Feature / new behaviour**

1. Invoke `grilling`. **Automatically** - do not wait to be asked, and do not tell the user to type
   `/grill-with-docs`. Keep grilling until the user tells you to stop. Their "stop" ends it; your
   own judgement that the design is now clear does not.
2. Invoke `domain-modeling` **only if** the ticket introduces a domain term or changes what an
   existing one means. Routine work does not move the domain model, and ADRs nobody needed are how
   ADRs stop being read.
3. Then `superpowers:writing-plans`.

> **Why not `/grill-with-docs`:** it is `disable-model-invocation: true`, so nothing but the user
> can trigger it. Its entire body is a call to `grilling` and `domain-modeling`, so invoking those
> two directly loses nothing. The user may still type it if they prefer.

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

**Branch first, before any code exists.** Cut it the moment the user says to start, so the default
branch stays clean and no work has to be rescued off it later.

- Use the overlay's naming convention. If it has none, use `<type>/<ticket>-<slug>`, for example
  `feat/PROJ-214-token-budget`, and say which you used.
- Branch from an up-to-date default branch.
- This is the **only** point in the workflow authorised to create a branch.

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

## Phase 3 - Review gate, then draft PR

Cheap correctness first, so no tokens are spent reviewing code that does not compile.

1. **Discover this project's gate.** Take it from the overlay. Only if there is no overlay, check
   `package.json` scripts, Makefile, CI config, or the project's CLAUDE.md for a pre-push /
   check-all script. Do not invent a command.
2. **Fast checks** - unit tests and typecheck for the affected package only.
3. **Quality pass** - the BUILT-IN `/code-review`, `medium` by default. Escalate to `high` or
   `max` only for auth, data-layer, security, or cross-package changes.
4. **Architecture pass** - ask the user to type `/improve-codebase-architecture`
   (`disable-model-invocation: true`, you cannot trigger it - and unlike `grill-with-docs` it has
   a real body, so there is nothing to call directly instead). Worth it on sprawling or
   structural diffs, not on every task.
5. **Apply fixes.**
6. **Full gate** - run the project-wide check from step 1. **It must pass before step 7.** A red
   gate ends the phase here: report it and stop. Do not open a PR on work you know is broken.
7. **Commit, push, open a draft PR.**
   - Stage by explicit path. Never `git add .` - repos may track `.env*` files holding live keys.
   - Push the ticket branch. **Never push to the default branch.**
   - `gh pr create --draft`, titled from the ticket.
   - **Body:** use the repo's own template if there is one. Check
     `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, and
     `.github/PULL_REQUEST_TEMPLATE/`. Fill every section it defines; do not drop the ones that
     are awkward to answer. Only if the repo has no template, use the fallback below, and say
     that you did.
   - **Draft, always.** Do not mark it ready for review, do not request reviewers, do not merge,
     do not enable auto-merge. The draft is the handover, not the decision.
8. **Stop.** Report the PR URL, what passed, and what did not, with the actual output.

### Fallback PR body

Only when the repo defines no template of its own:

```markdown
## Ticket
<number> - <link>

## What changed
<the diff in plain language, not a list of files>

## Why
<the problem, not the solution restated>

## How it was verified
<commands actually run, and their result>

## Risk and blast radius
<what breaks if this is wrong, and what it touches>

## Not covered
<what was deliberately left out, and why>
```

`Not covered` is the section that earns the template. A PR that omits it reads as complete when it
is not.

---

## Guardrails

- **Never start Phase 2 automatically.** Phase 1 ends with a plan handed over, then you wait for
  an explicit instruction to implement. The user starts the work, every time.
- **Commit, push and PR are authorised in Phase 3 only**, on the ticket's own branch, as a draft,
  and only after the full gate passes. That authorisation does not extend to the default branch,
  to marking a PR ready, to merging, or to any other point in the session.
- **Branches are created in Phase 2 only.** Never switch branches without explicit instruction.
- **Never `git add .`** - stage by explicit path.
- Report failures faithfully, with output. Never claim completion on unverified work.

### Name collision: `code-review`

Two skills share this name and take incompatible arguments:

- **built-in `/code-review`** - effort `low|medium|high|xhigh|max`, `--fix`, `--comment`;
  reviews the current diff. This is the one step 3 means.
- **mattpocock `code-review`** - takes a commit/branch/tag to diff *since*, reviews Standards
  and Spec in parallel subagents, and does **not** accept effort levels. Use it deliberately
  for branch-vs-main reviews, never as a drop-in for step 3.
