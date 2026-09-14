# claude-skills

A development workflow for [Claude Code](https://claude.com/claude-code), packaged as installable
skills: stress-test the plan, implement it test-first through subagents, then gate it behind a
real review before anything gets committed.

Two skills:

| Skill | What it is |
|---|---|
| `dev-flow` | The workflow itself. Project-agnostic. |
| `project-overlay` | A template for the per-project half: your real commands, your rules, your traps. |

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

1. PLAN       feature → /grill-with-docs → superpowers:writing-plans
              bug     → systematic-debugging → superpowers:writing-plans
              ⇒ a plan file on disk

              ⇣ STOP. You decide when implementation starts.

2. IMPLEMENT  superpowers:subagent-driven-development
              each task → its own subagent, TDD inside, returns a summary
              main thread stays lean

3. REVIEW     fast checks → /code-review → /improve-codebase-architecture
              → your project's full gate → you commit
```

Three properties that are load-bearing, and the reasons they are there:

**The plan goes on disk.** A long single session hits context compaction. Anything that exists
only in the transcript is on a timer; the plan file is what survives it.

**Implementation is delegated, not inline.** Each subagent's context is discarded when it returns,
so the main thread stays small across a long session. This is what makes one-session work
survivable, more than model choice does.

**The agent stops between phase 1 and phase 2, and never commits.** Both transitions are yours.
Approving a plan approves the plan, not the start of the work.

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
`improve-codebase-architecture` are both in this category, which is why the workflow above asks
*you* to type them.

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
