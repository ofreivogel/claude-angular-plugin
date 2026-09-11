---
name: nx-sync
description: Regenerates the Nx fork of the Angular skills under nx/ from the upstream skills, applying the translation rules in nx-rules.md. Use after pulling upstream changes, after editing nx-rules.md, or when nx/ has drifted from the upstream skills.
allowed-tools: Read, Write, Edit, Grep, Glob, WebFetch, Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(cp:*), Bash(find:*), Bash(grep:*), Bash(cmp:*), Bash(diff:*)
---

# nx-sync

Maintains `nx/` — the Nx fork of the upstream Angular skills — as reviewable repository content.

**`nx/` is written by this skill, but it is not a build artifact.** It is committed, read and
reviewed like any other file. Every change you make there must survive a human reading the diff.

## Layout

| Path | Role |
| :-- | :-- |
| `angular-developer/`, `angular-new-app/` | Upstream, **never edit** |
| `nx/` | The Nx fork: same files, Angular CLI translated to Nx |
| `nx-rules.md` | How to translate. The only place that defines this |

## Scope

Translate Angular-specific tooling only. Generic Nx knowledge — project graph, `affected`,
caching, library architecture, module boundaries, generator discovery — is **out of scope**; it
is covered by the Nx MCP server and the official Nx skills. Do not add such content, and do not
let the fork grow beyond a translation of the upstream skills.

## Procedure

### 1. Establish the starting point

Report what changed upstream since the last sync, so the work is bounded:

```
git log --oneline -5 -- angular-developer angular-new-app
git diff --stat HEAD -- angular-developer angular-new-app
```

For each file, compare upstream against its counterpart in `nx/` (`cmp -s`) to see which are
still byte-identical and which carry translations.

### 2. Verify the rules against the documentation — before writing anything

1. Read `nx-rules.md`. Load the source pages it lists and check the rules you are about to apply:

- Does every `@nx/angular:*` generator, executor and builder you would write still exist?
- Are any of them deprecated or removed?
- Do the flags still have those names and accepted values?
- Check names against `generators.json` and `executors.json` in the Nx repository, not only against the rendered documentation pages: the `/executors` page omits the `builders` section, so a real name such as `@nx/angular:dev-server` looks missing there.
- If the documentation is unreachable ask the first to continue or to aboart.

2. on failures in `nx-rules.md` provide correction and let confirm the changes `nx-rules.md.


### 3. Copy and translate

precondition: 3. **Correct `nx-rules.md` first, then translate.** 

Copy `angular-developer/` and `angular-new-app/` into `nx/`, then translate every file that
mentions the Angular CLI:

- **Part A** of `nx-rules.md` for the one-to-one replacements.
- **Part B** for everything that has no direct equivalent. These are instructions, not
  substitutions — the affected sections are rewritten, sometimes removed.

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
