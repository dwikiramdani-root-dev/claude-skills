---
name: project-overlay
description: Template for a per-project companion to dev-flow - the real gate commands, where this project's rules live, and its known traps. Copy this into a project's .claude/skills/ and fill it in.
---

# Project Overlay (template)

`dev-flow` is deliberately project-agnostic: it knows the shape of the work, not your commands.
This is the other half. Copy it to `<your-repo>/.claude/skills/<name>-flow/SKILL.md`, fill in the
placeholders, and delete anything that does not apply.

**Why two skills instead of one.** A workflow with `pnpm run pre-push` baked in is wrong in a Go
repo. A workflow with no commands at all makes the agent guess, and it guesses from the code next
door, which is often the code you are trying to stop it copying. Splitting them means the generic
half travels and the specific half is exact.

Rename the skill after your project (`acme-flow`, `api-flow`). Do not call it `project-overlay`,
or it will collide with this template on machines that have both.

---

## 1. Where this project's rules live

> Replace with the real paths. Be specific: an agent that cannot find your rules will infer them
> from nearby code.

- Conventions and architecture rules: `<path, e.g. CLAUDE.md, AGENTS.md, docs/architecture/>`
- If more than one source exists, say **which wins** when they disagree.

**Known stale or wrong documentation.** List anything in those docs that is currently incorrect,
and what the truth is. This is the highest-value section: contradictions are worse than gaps,
because the agent picks one and then defends it.

> Example shape: "`<file>` line N says X. That is stale; the real rule is Y. Follow Y."

---

## 2. Do not pattern-match on neighbouring code

> Delete this section only if your codebase genuinely has no rule violations.

Most codebases contain code that predates their current standards. List the known violations so
the agent treats them as exceptions rather than examples:

> Example shape: "N call sites still use `<forbidden pattern>`; they are legacy, not a template."

Counting them is worth the five minutes. "Some files do X" gets ignored; "27 files do X and all
of them are wrong" does not.

---

## 3. The real gate commands

> The commands, exactly as they run in this project. No invented flags.

**Fast checks** (the affected package only, for quick feedback):
```
<command>
```

**Full gate** (what CI runs, or the closest local equivalent):
```
<command>
```
Say what it covers, so the agent knows what is and is not verified by it.

**End-to-end / integration**, plus any setup they need first (a database, emulators, a dev
server):
```
<command>
```

---

## 4. Ticket, branch and PR conventions

> `dev-flow` asks for a ticket up front, cuts a branch before Phase 2, and opens a **draft** PR at
> the end of Phase 3. All three need project-specific answers. Without them the agent invents a
> branch name and a PR shape, and both are then wrong in a way nobody notices until review.

**Ticket system.** Which one, and the URL shape so a bare number can be turned into a link:

```
<e.g. Linear, https://linear.app/acme/issue/PROJ-214>
```

**Branch naming.** The real convention, with a real example:

```
<e.g. feat/PROJ-214-short-slug, fix/PROJ-98-short-slug>
```

Say what the type prefixes are, and whether the ticket number is required. If branches are cut
from something other than the default branch, say so and name it.

**PR template.** Where it lives, if it exists:

```
<e.g. .github/pull_request_template.md, or "none - use the dev-flow fallback">
```

If any section of that template is routinely left blank in this project, say which and why -
otherwise the agent will fill it in with something invented rather than leave it empty.

**Does this project even use PRs?** If work goes straight to a shared branch, say so explicitly
and say who is allowed to do it. Note any branch protection or ruleset, and whether it actually
blocks or merely warns - a rule that warns and then allows the push teaches the agent that the
rule is decorative.

---

## 5. Project guardrails

> Anything that would cause real damage. Be concrete about the blast radius.

Common ones worth stating explicitly if they apply:

- Files that are tracked but hold secrets, so `git add .` is unsafe. Name the staging rule.
- Deploy targets, and which ones require confirmation first.
- Whether this skill is tracked in git, and therefore whether it is shared with the team.
- Whether anything in this file is internal and should not be copied into a public repo.

---

## Checklist before you commit this

- [ ] Renamed the skill and its `name:` frontmatter to something project-specific
- [ ] Every `<placeholder>` replaced or the section deleted
- [ ] Every command actually run once, not written from memory
- [ ] Branch convention copied from a branch that exists, not from the style guide
- [ ] Branch protection checked by trying it, not by reading the settings page
- [ ] Nothing in it would be a disclosure if the repo became public
