---
name: rtk
description: Use when the user types /rtk, or before sweeping an unfamiliar codebase with reads and searches on a machine where the rtk CLI is installed.
---

# rtk

`rtk` ("Rust Token Killer") is a CLI proxy that compresses command output before you read it.
Installed with `rtk init -g`, it adds a PreToolUse hook that rewrites Bash commands transparently.

**The hook only sees Bash.** `Read`, `Grep` and `Glob` never pass through it. No hook can make you
*choose* the cheaper tool. That choice is this skill's only job.

---

## Preflight

```
command -v rtk
```

If it is missing, say so once, give the install line, and then follow everything below except the
rtk commands. The narrowing/committing rule is worth keeping on its own.

- macOS / Linux: `brew install rtk`
- Windows: `winget install rtk-ai.rtk`, plus `winget install BurntSushi.ripgrep.MSVC` - some
  filters shell out to `rg` and warn without it.

Then `rtk init -g` and restart Claude Code. Do not report the hook as working until a rewritten
command has actually run.

---

## The rule

Split on **what you are about to do with the text**, not on which tool is nicer.

### Narrowing - you do not yet know which file or line matters

Use rtk freely. Lossy is *correct* here: you are discarding candidates, and whatever survives gets
read properly in the next step. This is where the saving actually lives, because orientation is the
bulk of the reading.

| Task | Command |
|---|---|
| What is in this tree | `rtk ls .` |
| Where is X mentioned | `rtk grep "X" .` |
| Which files match | `rtk find "*.ts" .` |
| What shape is this file | `rtk read <path> -l aggressive` |
| Two-line gist | `rtk smart <path>` |
| Condensed diff | `rtk git diff` |

### Committing - you are about to edit, quote, or assert exact contents

Read the real bytes: the built-in `Read` with `offset`/`limit`, or `sed -n '120,180p'`. Ranged reads
keep this cheap, so the rule costs far less than it looks.

**The switch fires the moment a file becomes an edit target or a quotation source.** Not when it
feels important enough.

### Verbose command output - tests, builds, lint, git

The hook already compresses these; you do not need to do anything. Your one job is recovery: when a
failure's detail was filtered away, run `rtk recall <hash>`. Do not re-run the command unfiltered.

---

## Traps

None of these produce an error message.

**`Edit`'s read-precondition does not verify that you saw the text.** It tracks that the file was
read at some point, not that its contents are in your context. Verified: read only line 1 of a
four-line file, then edit line 3 - it succeeds and reports success. So an `old_string` assembled
from `rtk read -l aggressive` output, where function bodies are stripped, may be something you
inferred rather than read, and nothing will stop you.

**Filters are lossy by design.** Test output is failures only. A test that passes for the wrong
reason is invisible.

**The advertised saving is output reduction, not bill reduction.** rtk's own README: the reduction
"dilutes at every step." Bash output is one contributor to input tokens among several.

**Correctness beats frugality, always.** A wrong edit costs a debugging cycle that dwarfs any read
this skill could have saved. When the two conflict, spend the tokens.

---

## Rationalisations

| Excuse | Reality |
|---|---|
| "I only need the signature to write this edit" | `old_string` must match bytes you have actually seen. A signature is not a body. |
| "The file is small, `rtk read` gave me basically all of it" | "Basically" is the failure mode. Read it. |
| "`Edit` accepted it, so my view must have been current" | `Edit` does not check that. See Traps. |
| "Re-running the test shows me more than `recall` would" | And costs the full output plus the run. Use `recall`. |
| "This file is obviously unchanged since I skimmed it" | Then a ranged read costs almost nothing. Do it. |
| "Using the built-in `Read` wastes the whole point of rtk" | The point is cheap *narrowing*. Committing was never in scope. |

## Red flags - stop and read the bytes

- Writing an `old_string` you have not seen in full
- Quoting a file you only ran `rtk smart` or `rtk read -l aggressive` on
- Saying a test passes when you only saw filtered output
- Reaching for rtk on a file you have already decided to change

---

## Overlay block

`/rtk` only helps on sessions where it is invoked. To make the discipline always-on for a project,
paste this into the rules section of that project's overlay skill (see `project-overlay`):

```markdown
### Reading and searching

`rtk` is installed here and its Bash hook is active. Narrow with rtk (`rtk ls`, `rtk grep`,
`rtk find`, `rtk read -l aggressive`); read exact bytes before any edit or quotation, with a
ranged built-in `Read` or `sed -n`. Never build an `old_string` from filtered output - `Edit`
will accept it without checking. On a filtered failure, `rtk recall <hash>` rather than re-running.
```
