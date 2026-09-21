# claude-skills

A development workflow for [Claude Code](https://claude.com/claude-code), packaged as installable
skills: stress-test the plan, implement it test-first through subagents, then gate it behind a
real review before anything gets committed.

Three skills:

| Skill | What it is |
|---|---|
| `dev-flow` | The workflow itself. Project-agnostic. |
| `project-overlay` | A template for the per-project half: your real commands, your rules, your traps. |
| `rtk` | Tool-choice discipline for [rtk](https://github.com/rtk-ai/rtk), so a session reads cheaply without reading wrongly. |

## Prerequisites

**These are required, not suggested.** `dev-flow` delegates to skills from two other sources, and
without them it points at things that are not there.

```bash
claude plugin install superpowers@claude-plugins-official
```

```bash
npx skills@latest add mattpocock/skills
```

> **Heads up on the second one.** `--skill=<name>` does *not* scope the install. It pulls the
> entire repo (37 skills at time of writing) into `~/.agents/skills/`, symlinked into
> `~/.claude/skills/`. Prune what you do not want by removing symlinks from `~/.claude/skills/`;
> the originals stay in `~/.agents/skills/` and relink with `ln -s`.

## Install

```bash
npx skills@latest add dwikiramdani-root-dev/claude-skills
```

Then type `/dev-flow` in Claude Code, or just start a feature and it will pick it up.

## The workflow

```
ONE SESSION

0. INTAKE     full ticket or your own description? + ticket number and link

1. PLAN       feature → grilling (automatic, ends when you say stop)
                      → domain-modeling (only if the domain model moves)
              bug     → systematic-debugging
              both    → superpowers:writing-plans
              ⇒ a plan file on disk

              ⇣ STOP. You decide when implementation starts.

2. IMPLEMENT  branch first, then superpowers:subagent-driven-development
              each task → its own subagent, TDD inside, returns a summary
              main thread stays lean

3. REVIEW     fast checks → /code-review → /improve-codebase-architecture
              → your project's full gate, which must pass
              ⇒ commit, push, draft PR. Never ready, never merged.
```

Three properties that are load-bearing, and the reasons they are there:

**The plan goes on disk.** A long single session hits context compaction. Anything that exists
only in the transcript is on a timer; the plan file is what survives it.

**Implementation is delegated, not inline.** Each subagent's context is discarded when it returns,
so the main thread stays small across a long session. This is what makes one-session work
survivable, more than model choice does.

**The agent stops between phase 1 and phase 2, and stops again at a draft PR.** Approving a plan
approves the plan, not the start of the work. Phase 3 may commit, push and open a PR, but only on
the ticket's own branch, only once the full gate is green, and only as a draft. Marking it ready,
merging, and anything touching the default branch stay yours.

## The two-layer idea

This is the part worth stealing even if you ignore the rest.

A workflow with `pnpm run pre-push` baked into it is wrong in a Go repo. A workflow with no
commands at all makes the agent guess, and it guesses from the code next door, which is often
exactly the code you are trying to stop it copying.

So: `dev-flow` is installed once per machine and knows the *shape* of the work. `project-overlay`
is copied into each repo's `.claude/skills/` and holds that project's real commands, where its
rules live, and its known traps. The overlay wins wherever they disagree.

The highest-value section of an overlay is usually the list of things your own documentation gets
wrong. A contradiction is worse than a gap: the agent picks one side and then defends it.

## Reading cheaply (`rtk`)

Optional, unlike the prerequisites above: the skill degrades to useful advice without it.

[rtk](https://github.com/rtk-ai/rtk) compresses command output before the agent reads it, and
`rtk init -g` installs a hook that rewrites Bash calls for you. The hook is the easy half and it
needs no skill. The hard half is that the hook **only sees Bash** - `Read`, `Grep` and `Glob` go
straight past it, and nothing can make an agent *choose* the cheaper tool.

So the `rtk` skill is one rule: narrow with rtk, commit with exact reads. Lossy output is right
while you are still discarding candidates and wrong the moment you are writing an edit. That line
matters more than it sounds, because `Edit`'s "read the file first" guardrail does not check that
you saw the text - only that the file was read at some point. An `old_string` inferred from
stripped-out function bodies is accepted without complaint.

Type `/rtk`, or paste the skill's overlay block into a project overlay to make it always-on.

## Traps worth knowing about

Found the hard way. None of these produce an error message.

**`code-review` is an ambiguous name.** Claude Code ships a built-in `/code-review` taking effort
levels (`low`…`max`), `--fix` and `--comment`. `mattpocock/skills` ships a different
`code-review` taking a commit/branch/tag to diff *since*, which rejects effort levels entirely.
Installing the second shadows the first. Rename one if you use both.

**Permission entries pin plugin versions.** Approvals recorded against
`.../superpowers/6.1.0/...` silently stop matching when 6.2.0 installs, and every previously
approved script starts prompting again. Prefer a prefix rule such as
`Bash(bash /path/to/scripts/task-brief *)` over the exact invocations Claude Code records for you.

**Approvals accumulate as dead weight.** Each one is recorded with the exact arguments used, so a
plan file name or a commit SHA gets baked in and can never match again. They are worth pruning.

**Some skills can only be typed by a human.** Skills with `disable-model-invocation: true` in
their frontmatter cannot be triggered by Claude, only by you. `grill-with-docs` and
`improve-codebase-architecture` are both in this category.

But open them before you accept the gate. `grill-with-docs` is a two-line wrapper whose entire
body is a call to `grilling` and `domain-modeling` — and neither of *those* is gated. So
`dev-flow` calls them directly and gets the same interview automatically, without editing anyone
else's skill. `improve-codebase-architecture` has a real body, so it genuinely has to be typed.
A gate on a wrapper is only as strong as the things it wraps.

**Skill descriptions cost context on every session.** They are all loaded at startup, in every
project. Installing a large skills repo and keeping all of it is not free.

## Contributing

Issues and PRs welcome, particularly:

- Overlay examples for stacks other than TypeScript
- Traps like the ones above, with a reproduction
- Places where `dev-flow` assumes something it should not

Keep `dev-flow` project-agnostic. Anything specific to one stack belongs in an overlay.

## License

MIT. See [LICENSE](LICENSE).
