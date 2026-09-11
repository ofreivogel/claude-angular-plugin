# Nx Translation Rules

Reference used by the `/nx-sync` skill to translate the upstream `angular-developer` skill into
its Nx counterpart under `nx/`.

Three parts: **A** the one-to-one replacements, **B** the cases with no direct Nx equivalent,
**C** what is deliberately not forked at all.

**This file is the only place that defines how a translation is done.** Keeping it here —
rather than in the skill prompt — is what makes repeated syncs consistent.

## Scope

Only Angular-specific tooling is translated. Generic Nx knowledge (project graph, `affected`,
caching, library architecture, module boundaries, generator discovery) is **out of scope** — it
is covered by the Nx MCP server (`npx nx mcp`) and the official Nx skills (`nx-workspace`,
`nx-generate`, `nx-run-tasks`, `nx-plugins`, installed via `npx nx configure-ai-agents`).
Do not add such content to the Nx fork of the skills.

## Verification sources

Every rule below carries a source. Before applying a rule, the skill loads the corresponding
page and confirms the rule still holds. Rules that no longer match the documentation are
corrected **here first**, then applied.

| Key | URL |
| :-- | :-- |
| `/introduction` | https://nx.dev/docs/technologies/angular/introduction |
| `/generators` | https://nx.dev/docs/technologies/angular/generators |
| `/executors` | https://nx.dev/docs/technologies/angular/executors |
| `/migrations` | https://nx.dev/docs/technologies/angular/migrations |
| `/create-workspace` | https://nx.dev/docs/reference/create-nx-workspace |
| `generators.json` | https://raw.githubusercontent.com/nrwl/nx/master/packages/angular/generators.json |
| `executors.json` | https://raw.githubusercontent.com/nrwl/nx/master/packages/angular/executors.json |

The two JSON files are authoritative and should be preferred when a name is in doubt. The
`/executors` documentation page renders only the `executors` section of `executors.json` and
**omits the `builders` section** — `@nx/angular:dev-server` is real but does not appear there.
Checking a name against the docs page alone produces false negatives.

Last verified: 2026-09-11 against Nx 23.2. `@nx/angular` requires Angular >= 20 < 23.

## How the "no `ng` commands left" check is applied

The check is zone-aware, because not every `ng` token is a command to translate:

- **Inside a fenced code block** — a leftover `ng <verb>` is an **error**. Nothing an agent might
  copy and run may remain.
- **In prose or an inline code span** — a leftover is **reported, not rejected**. A deliberate
  negative mention ("not `ng update`", "do not use the Angular CLI here") is valuable guidance and
  is kept. Each such occurrence must be confirmed as intentional during review.
- **Angular identifiers** (rule A4) are never commands and are matched separately by count.

Known churn to re-check on every sync — these changed recently and will change again:

- `@nx/jest:jest` is deprecated and is removed in Nx v24 (`nx g @nx/jest:convert-to-inferred`).
- `@nx/vite:test` no longer exists; Vitest lives in `@nx/vitest`.
- `@nx/angular:move` and `@nx/angular:ngrx` were removed in Nx 23
  (`@nx/workspace:move`, `ngrx-root-store` / `ngrx-feature-store`).
- `@nx/angular:unit-test` requires Angular >= 21.

---

# Part A — one-to-one replacements

## A1 Generators

Nx-owned generators take the **path as a positional argument**. There is no `--project`.

| Angular CLI | Nx |
| :-- | :-- |
| `ng g c foo` | `nx g @nx/angular:component apps/my-app/src/app/foo/foo` |
| `ng g d foo` | `nx g @nx/angular:directive apps/my-app/src/app/foo/foo` |
| `ng g p foo` | `nx g @nx/angular:pipe apps/my-app/src/app/foo/foo` |
| `ng g application foo` | `nx g @nx/angular:application apps/foo` |
| `ng g library foo` | `nx g @nx/angular:library libs/foo` |

Verified generator list in `@nx/angular` (`/generators`): `application` (`app`), `component` (`c`),
`component-test`, `convert-to-application-executor`, `convert-to-rspack`, `convert-to-with-mf`,
`cypress-component-configuration`, `directive` (`d`), `federate-module`, `host` (`consumer`),
`library` (`lib`), `library-secondary-entry-point`, `ngrx-feature-store`, `ngrx-root-store`,
`pipe` (`p`), `remote` (`producer`), `scam`, `scam-directive`, `scam-pipe`, `scam-to-standalone`,
`setup-mf`, `setup-ssr`, `stories`, `storybook-configuration`, `web-worker`.

**Never write an `@nx/angular:<name>` that is not on that list.** See rule B7.

## A2 Tasks

| Angular CLI | Nx |
| :-- | :-- |
| `ng build` | `nx build <project>` |
| `ng serve` | `nx serve <project>` |
| `ng test` | `nx test <project>` |
| `ng lint` | `nx lint <project>` |
| `ng build --configuration=x` | `nx build <project> --configuration=x` |
| `ng add X` | `nx add X` |
| `ng version` | `nx report` |

The project argument is mandatory — see rule B2.

## A3 Terminology

| Angular CLI | Nx |
| :-- | :-- |
| builder | executor |
| schematic | generator |
| `angular.json` | `project.json` per project + `nx.json` (rule B5) |

## A4 Identifiers that must NOT be touched

These are Angular template and API syntax, not CLI invocations. A translation that changes
them is a defect, and the counts must match the upstream file exactly:

`ng-template` · `ng-content` · `ng-container` · `ngModel` · `ngSubmit` · `ngIf` · `ngFor` ·
`ngClass` · `ngSrc` · `ng-deep` · `ng-valid` · `ng-invalid` · `ng-touched` · `ng-untouched` ·
`ng-pristine` · `ng-dirty` · `ng-submitted` · `NgModule` · `ng-packagr`

**Not** on this list — these are Angular CLI packages that legitimately disappear under Nx,
because e2e setup moves to `--e2eTestRunner` (rule B2): `playwright-ng-schematics`,
`@cypress/schematic`, `@nightwatch/schematics`, `@wdio/schematics`, `@puppeteer/ng-schematics`.
A drop in their count is expected, not a defect.

---

# Part B — rules for cases with no one-to-one mapping

Each rule: **Trigger** · **Action** · **Source / Check**.
If none of these rules covers a case, **do not invent a translation** — flag it in the report and
propose a new rule for this file.

## B1 — task commands without a project

**Trigger:** `ng build`, `ng serve`, `ng test`, `ng lint` with no project.

**Action:** add the project argument: `nx build <project>`. Where the upstream text left the
project implicit ("run the build"), make the placeholder visible. For running a target across
projects use `nx run-many -t <target>`; the explicit long form is `nx run <project>:<target>:<configuration>`.

`nx build` with no project fails in a monorepo — a bare replacement produces a command that is
guaranteed to break.

**Source:** `/executors` — **Check:** no `nx <target>` without a following project argument or flag.

## B2 — `ng e2e` and e2e setup

**Trigger:** `ng e2e`, and the `ng add <framework>-schematics` block in `references/e2e-testing.md`.

**Action:** in Nx, e2e is a **separate project**: `nx e2e <app>-e2e`, living in `apps/<app>-e2e`.
It is created together with the application (`--e2eTestRunner=playwright|cypress|none`,
Playwright being the generator default), not added afterwards with `ng add`.

`@nx/cypress` is deprecated as of Nx 23 — prefer Playwright for new setups and say so.

**Source:** `/generators`, `/executors` — **Check:** target project name ends in `-e2e`; no
`ng add …-schematics` remains.

## B3 — `ng g environments`

**Trigger:** `ng generate environments`.

**Action:** `@nx/angular` has no `environments` generator, but Angular's own schematic passes
through — rule B6 applies:

```bash
nx g @schematics/angular:environments --project=my-app
```

`project` is required (`$source: projectName`). The schematic creates the environment files and
registers the `fileReplacements`; under Nx that lands in the project's `project.json`.

Also mention `fileReplacements` explicitly, since a reader coming from the Angular CLI will look
for it in `angular.json`.

**Source:** `@schematics/angular` `collection.json` and `environments/schema.json` — **Check:**
the generator call is present and carries `--project`.

## B4 — `ng deploy`

**Trigger:** `ng deploy`, and the `ng add @angular/fire` + deploy flow in `references/cli.md`.

**Action:** there is no generic Nx deploy command and no deploy executor in `@nx/angular`.
Either drop the section or mark it explicitly as depending on the deployment plugin in use.
**Never write `nx deploy`** as though it were a built-in.

**Source:** `/executors` — **Check:** `nx deploy` does not appear in the output.

## B5 — `angular.json`

**Trigger:** any mention of `angular.json`, in code or in prose.

**Action:** Nx has no `angular.json`. Per-project configuration lives in `project.json`;
workspace-wide defaults (`targetDefaults`, `namedInputs`, `plugins`, generator defaults) live in
`nx.json`. `nx init` on an existing Angular CLI workspace splits `angular.json` into one
`project.json` per project.

Many Nx projects have **no explicit `targets` block at all**, because plugins such as
`@nx/angular/plugin` infer targets from the tool configuration. `nx show project <name>` prints
the resolved configuration.

**Source:** `/introduction` — **Check:** `angular.json` no longer appears in the output.

## B6 — generators with no `@nx/angular` counterpart

**Trigger:** `ng g service|guard|resolver|interceptor|module|environments|class|interface|enum`.

**Action:** `@nx/angular` provides **no** generator for these. Pass Angular's own schematic
through, which is where `--project` is correct — it is `required` in every one of these schemas,
with `$default: {$source: projectName}`:

```bash
nx g @schematics/angular:service my-data --project=my-app
nx g @schematics/angular:guard auth --project=my-app
nx g @schematics/angular:environments --project=my-app
```

Verified members of `@schematics/angular` relevant here: `service` (`s`), `guard` (`g`),
`resolver` (`r`), `interceptor`, `module` (`m`), `environments`, `class` (`cl`), `interface`
(`i`), `enum` (`e`).

**Do not invent `@nx/angular:service` or similar** — and do not conclude from "`@nx/angular` has
no X generator" that X cannot be generated. Check `@schematics/angular` before declaring a gap.

**Source:** `@schematics/angular` `collection.json` — **Check:** every `@nx/angular:*` name in
the output appears in the verified generator list of rule A1.

## B7 — `--project` on an Nx generator

**Trigger:** an `@nx/angular:*` invocation carrying `--project`.

**Action:** there are **three** scoping conventions, and mixing them produces commands that do
not run. Verified against the schemas:

| Family | Scoping | Verified in the schema |
| :-- | :-- | :-- |
| `@nx/angular:*` generators | **positional path**, no `--project` | `path` / `directory` is `required` with `$default: {$source: argv, index: 0}`; there is no `project` property at all |
| `@schematics/angular:*` generators | `name` positional **+ `--project`** | both `name` and `project` are `required`; `project` defaults to `$source: projectName` |
| `@angular/core:*` migrations | **`--path <dir>`**, no `--project` | the schemas expose only `path` (plus migration-specific options); none has a `project` property |

So `nx g @nx/angular:component apps/app/src/app/foo/foo --project=app` is wrong (unknown flag),
and so is `nx g @angular/core:control-flow --project=app`. The upstream text claims these
migrations accept `--project`; that is incorrect and must not be carried over.

**Source:** the generator `schema.json` files — **Check:** no `@nx/angular:*` and no
`@angular/core:*` invocation carries `--project`; every `@schematics/angular:*` invocation does.

## B8 — `ng new`

**Trigger:** `ng new`, and the workspace-creation decision tree in `angular-developer/SKILL.md`.

**Action:** use `create-nx-workspace`:

```bash
npx create-nx-workspace@latest <name> --preset=angular-monorepo --aiAgents=claude
```

Verified preset names: `angular`, `angular-monorepo`, `angular-standalone`. Verified flags:
`--preset`, `--style`, `--ssr`, `--unitTestRunner`, `--e2eTestRunner=playwright|cypress|none`,
`--bundler`, `--appName`, `--packageManager=npm|yarn|pnpm|bun`, `--nxCloud`,
`--aiAgents=claude|codex|copilot|cursor|gemini|opencode|none`, `--interactive`.

`--aiAgents=claude` is the counterpart of the Angular CLI's `--ai-config=claude`; it also wires
up the Nx MCP server and the official Nx skills. `npx nx configure-ai-agents` does the same for
an existing workspace.

To convert an existing Angular CLI workspace: `npx nx@latest init`, or
`npx nx@latest init --integrated` for the full monorepo layout.

**Source:** `/create-workspace`, `/introduction` — **Check:** every flag and preset named in the
output appears in the create-nx-workspace reference.


## B9 — test commands

**Trigger:** `ng test`, and the command lines in the testing references.

**Action:**

```bash
nx test <project>
nx test <project> --watch
nx test <project> --coverage
```

Running a single file or filtering by name is **runner-dependent** — name the form that matches
the configured runner, never a generic invented one:

- `@angular/build:unit-test` / `@nx/angular:unit-test` (Angular >= 21): `--include=<glob>`
  (default `["**/*.spec.ts","**/*.test.ts"]`), `--filter=<regex on suite/test names>`,
  `--coverage` (default false), `--watch` (defaults to true on a TTY).
- `@nx/jest:jest`: `--testFile=<path>`, `--testPathPatterns=<regex>`, `-t <name>`.
  Deprecated, removed in Nx v24.
- `@nx/vitest:test`: arguments after `--` are passed to the Vitest CLI.

The generator flag is `--unitTestRunner=vitest-angular|vitest-analog|jest|none`. On Angular >= 21
with the default esbuild bundler the default is `vitest-angular`.

**Source:** `/executors` — **Check:** every option named appears on the documented test executor.


## B10 — package, generator and executor names in general

**Trigger:** any `@nx/*` name in the output.

**Action:** verify it against the current documentation and mark deprecations explicitly.
Verified executor list in `@nx/angular` (`/executors`): `application`, `browser-esbuild`,
`delegate-build`, `extract-i18n`, `module-federation-dev-server`, `module-federation-dev-ssr`,
`ng-packagr-lite`, `package`, `unit-test`.

Verified builder list in `@nx/angular` (`executors.json`, `builders` section — these are
referenced in `project.json` under `"executor"` like any other): `webpack-browser`, `dev-server`,
`webpack-server`.

`dev-server` serves through webpack when the build target uses a webpack-based executor, and
through Vite when it uses an esbuild-based one.

**Source:** `executors.json`, `generators.json` (authoritative), `/executors`, `/generators` —
**Check:** the name exists in the JSON under either `executors` or `builders` and is not marked
removed.

---

# Part C — what is not forked

Three things upstream ships that this fork deliberately leaves out. They are listed as rules so a
sync does not quietly translate them back in. For each, note **what must not be lost** — dropping
a file is only correct if the Angular-specific part of it survives somewhere.

## C1 — the `angular-new-app` skill

**Trigger:** the skill directory itself.

**Action:** **do not fork it.** Creating a workspace is `create-nx-workspace`, which is generic Nx
territory and covered by the official Nx skills. The Nx plugin ships only `angular-developer`.

**Not to be lost:** a one-line pointer at `npx create-nx-workspace@latest <name>
--preset=angular-monorepo` in the *Creating Projects* section of `SKILL.md`, so the path stays
discoverable without duplicating the guidance.

**Check:** `nx/` contains no `angular-new-app/`; `nx/.claude-plugin/plugin.json` lists only
`./angular-developer`.

## C2 — `references/cli.md`

**Trigger:** the file itself.

**Action:** **drop it.** Once translated it described `nx g`, `nx build`, `nx serve`, `nx test`
and `nx lint` — which is Nx's own territory, covered by the Nx MCP server and the official Nx
skills (`nx-generate`, `nx-run-tasks`). Keeping it here would duplicate them and go stale.

What must **not** be lost is the Angular-specific part, which no Nx skill covers: that Nx-owned
generators take the path positionally, and that `@nx/angular` has no generator for services,
guards, resolvers, interceptors or modules so those pass `@schematics/angular` through with
`--project`. That guidance lives in the *Generating Angular Code* section of `SKILL.md`.

**Not to be lost:** exactly that section — the positional-path rule and the `@schematics/angular`
fallback (rules B6, B7). Without it the skill invents `@nx/angular:service`.

Remove the `cli.md` bullet from `SKILL.md` along with the file.

**Source:** — **Check:** `references/cli.md` absent from `nx/`; no dead link in `SKILL.md`; the
positional-path and `@schematics/angular` fallback rules still present in `SKILL.md`.

## C3 — `references/mcp.md`

**Trigger:** the file itself.

**Action:** **drop it.** It documents the MCP server bundled with the Angular CLI
(`devserver.start` runs `ng serve`, `list_projects` reads `angular.json`) — a server this plugin
deliberately does not ship, for a CLI that is usually absent in an Nx workspace.

Translating it to describe the Nx MCP server was the obvious move and is still wrong: that
server, its setup via `npx nx configure-ai-agents` and its tools are Nx's own territory, already
covered by the official Nx skills. A copy here would only go stale.

Note also that a verb substitution would be worse than dropping the file — it would turn a
visible gap into a false claim about tools this plugin does not provide.

Remove the `mcp.md` bullet from `SKILL.md` along with the file.

**Source:** `/introduction` — **Check:** `references/mcp.md` absent from `nx/`; no dead link in
`SKILL.md`.

## C4 — `references/migrations.md`

**Trigger:** the file itself.

**Action:** **drop it.** Version updates are `nx migrate` — a two-phase flow (`nx migrate latest`,
then `nx migrate --run-migrations` after installing) that belongs to Nx and is covered by the Nx
skills. `ng update` is not used in an Nx workspace at all.

The file also listed Angular's own code-modernization schematics (`control-flow`, `inject`,
`signal-input-migration`, `signal-queries-migration`, `output-migration`, `self-closing-tag`, and
the three-phase `standalone` workflow). Those are dropped with it by decision, not by oversight.

**Not to be lost:** nothing is carried over. If the modernization schematics are wanted back
later, they belong in their own reference, and note that they are scoped with **`--path <dir>`
only** — none of them has a `project` option (rule B7).

Remove the `migrations.md` bullet from `SKILL.md` along with the file.

**Check:** `references/migrations.md` absent from `nx/`; no dead link in `SKILL.md`.
