# Automatic Migrations & Code Modernization

Two different things are called "migration" here, and they use different commands:

- **Version updates** — moving Nx, Angular and the plugins to newer versions. Use `nx migrate`.
- **Code modernization** — applying a single Angular refactoring schematic to existing code.
  Use `nx g @angular/core:<name>`.

Always prefer these automated schematics over manual text replacement.

## 1. Version Updates (`nx migrate`)

In an Nx workspace, do **not** run the Angular CLI's update command. Nx carries Angular's own
migrations alongside the Nx ones and keeps them in step; running them separately desynchronizes
the workspace.

`nx migrate` is deliberately two-phase: the first command only plans, the second applies. That
split is the point — it lets you review, reorder or drop individual migrations before they touch
your code.

```bash
npx nx migrate latest             # updates package.json versions, writes migrations.json, changes no code
npm install                       # or pnpm/yarn install
npx nx migrate --run-migrations   # applies the migrations
```

Between the two, open `migrations.json` and review it. Entries can be reordered or removed to
run migrations one at a time or to skip one.

Useful flags:

| Flag                             | Effect                                                      |
| :------------------------------- | :---------------------------------------------------------- |
| `--from` / `--to`                | Pin the version range explicitly                            |
| `--include=required\|optional\|all` | `required` keeps the change set small (Nx + plugins only)  |
| `--interactive`                  | Choose which migrations to include                          |
| `--createCommits`                | One commit per migration, which makes the result reviewable |

Upgrade **one major version at a time**. `nx report` shows the currently installed versions.

## 2. Code Modernization Schematics

To view the available schematics for the installed core framework version:

```bash
nx g @angular/core: --help
```

Apply a specific syntax update. These migrations are scoped with **`--path <dir>`** only — they
have no `--project` option, unlike the `@schematics/angular` generators:

| Feature to Modernize      | Command to Execute                                          |
| :------------------------ | :---------------------------------------------------------- |
| **Built-in Control Flow** | `nx g @angular/core:control-flow`                           |
| **Signal-based Inputs**   | `nx g @angular/core:signal-input-migration`                 |
| **Signal Queries**        | `nx g @angular/core:signal-queries-migration`               |
| **Functional Outputs**    | `nx g @angular/core:output-migration`                       |
| **`inject()` Function**   | `nx g @angular/core:inject`                                 |
| **Self-Closing Tags**     | `nx g @angular/core:self-closing-tag`                       |
| **Standalone**            | `nx g @angular/core:standalone` (See workflow below)        |

In a monorepo, point `--path` at a single project rather than letting a migration run across the
whole workspace, so each change set stays reviewable:

```bash
nx g @angular/core:control-flow --path=apps/my-app/src
```

## Specialized Workflow: Migrating to Standalone

The Standalone migration is an interactive, multi-step refactoring. You **MUST** perform this in
three discrete stages, verifying that the application builds and runs correctly after each stage
completes:

1. **Phase 1**: Run `nx g @angular/core:standalone --path=apps/my-app/src` and select the option
   to **Convert all components, directives, and pipes to standalone**.
2. **Phase 2**: Verify the build with `nx build my-app`. Run the command again and select
   **Remove unnecessary NgModule classes**.
3. **Phase 3**: Verify the build with `nx build my-app`. Run the final pass and select
   **Bootstrap the project using standalone APIs**.

After each phase, `nx affected -t build test` shows whether anything else in the workspace broke.
