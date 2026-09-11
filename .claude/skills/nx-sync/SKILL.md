---
name: nx-sync
description: Regenerates the Nx fork of the angular-developer skill under nx/ from the upstream skill, applying the translation rules in its own nx-rules.md reference. Use after pulling upstream changes, after editing nx-rules.md, or when nx/ has drifted from the upstream skills.
allowed-tools: Read, Write, Edit, Grep, Glob, WebFetch, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(cp:*), Bash(find:*), Bash(grep:*), Bash(cmp:*), Bash(diff:*)
---

# nx-sync

Maintains `nx/` — the Nx fork of the upstream `angular-developer` skill — as reviewable repository content.

**`nx/` is written by this skill, but it is not a build artifact.** It is committed, read and
reviewed like any other file. Every change you make there must survive a human reading the diff.

## Layout

| Path | Role |
| :-- | :-- |
| `angular-developer/` | Upstream, **never edit** |
| `nx/angular-developer/` | The Nx fork: same files, Angular CLI translated to Nx |
| `nx-rules.md` (next to this file) | How to translate. The only place that defines this |

## Scope

Translate Angular-specific tooling only. Generic Nx knowledge — project graph, `affected`,
caching, library architecture, module boundaries, generator discovery — is **out of scope**; it
is covered by the Nx MCP server and the official Nx skills. Do not add such content, and do not
let the fork grow beyond a translation of the upstream skill.

Four things are deliberately **not** forked, and a sync must not reintroduce them. **Part C** of
nx-rules.md states each one with what must not be lost along with it:

- the `angular-new-app` skill (C1) — workspace creation is `create-nx-workspace`, generic Nx
  territory. The Nx plugin ships only `angular-developer`.
- `references/cli.md` (C2) — translated it would describe `nx g`, `nx build`, `nx serve` and
  `nx test`, which the Nx skills already cover. Its Angular-specific remainder lives in the
  *Generating Angular Code* section of the skill's `SKILL.md`.
- `references/mcp.md` (C3) — the Nx MCP server, its setup and its tools are Nx's own territory.
- `references/migrations.md` (C4) — version updates are `nx migrate`, an Nx flow. Angular's
  code-modernization schematics went with it by decision; nothing is carried over.

## Procedure

### 1. Establish the starting point

Report what changed upstream since the last sync, so the work is bounded:

```
git log --oneline -5 -- angular-developer
git diff --stat HEAD -- angular-developer
```

For each file, compare upstream against its counterpart in `nx/` (`cmp -s`) to see which are
still byte-identical and which carry translations.

### 2. Verify the rules against the documentation — before writing anything

**1. Check the rules.** Read [nx-rules.md](nx-rules.md), load the source pages it lists, and
verify the rules you are about to apply:

- Does every `@nx/angular:*` generator, executor and builder you would write still exist?
- Are any of them deprecated or removed?
- Do the flags still have those names and accepted values?

Check names against `generators.json` and `executors.json` in the Nx repository, not only against
the rendered documentation pages: the `/executors` page omits the `builders` section, so a real
name such as `@nx/angular:dev-server` looks missing there.

**If the documentation is unreachable, ask the user whether to continue or abort.** Do not decide
this yourself, and do not guess at the content of a page you could not read.

**2. Propose corrections and have them confirmed.** Where a rule no longer matches the
documentation, show the proposed change to `nx-rules.md` — the old rule, the new one, and the
source that contradicts it — and **wait for confirmation before writing it**. Do not silently
rewrite the rules.

This is the failure mode the whole setup exists to catch: Nx moves, and a rule set that is never
re-checked quietly starts producing commands that do not run.

### 3. Copy and translate

> **Precondition:** `nx-rules.md` is corrected and the corrections are confirmed. Never translate
> against a rule you already know to be wrong.

Copy `angular-developer/` into `nx/`, then translate every file that mentions the Angular CLI:

- **Part A** of [nx-rules.md](nx-rules.md) for the one-to-one replacements.
- **Part B** for everything that has no direct equivalent. These are instructions, not
  substitutions — the affected sections are rewritten, sometimes removed.
- **Part C** lists what is not forked at all; do not copy those files.

Two hard rules:

- **Never invent a command.** If no rule covers a case, leave it, flag it in the report and
  propose a new rule for `nx-rules.md`.
- **Never "repair" a section by swapping verbs** when the underlying concept does not exist in
  Nx. That turns a visible gap into a false claim, and the checks will pass while the content is
  wrong.

Files with no Angular CLI reference are copied unchanged.

### 4. Check

1. No `ng <verb>` inside a fenced code block anywhere in `nx/`. Occurrences in prose or inline
   spans are reported, not rejected — a deliberate negative mention ("not `ng update`") is
   intentional and must be confirmed as such.
2. Every `@nx/*` generator, executor and plugin name, and every `nx <verb>`, was confirmed
   against the documentation in step 2.
3. Every `nx <target>` carries a project argument or an explicit flag — a bare `nx build` fails
   in a monorepo.
4. The protected Angular identifiers in rule A4 occur exactly as often in `nx/` as upstream.
5. Every relative link in `nx/**` resolves, and no link points outside its own skill directory.
6. `nx/` has the same file count as upstream, minus anything a rule deliberately removed.

### 5. Report

State plainly:

- which upstream files changed since the last sync,
- which rules were applied, and which had to be corrected against the documentation,
- cases with no rule, with a proposed rule for each,
- which checks passed and which did not.
- do not commit the changes so that the changes can be verified simple
