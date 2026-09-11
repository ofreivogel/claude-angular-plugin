> Dieses Repo ist ein fork des offziellen angular skill gepackt in ein angular code plugin
# Angular Skills

The Angular skills are designed to help coding agents create applications aligned with the latest versions of Angular, best practices, new features and manage Angular applications effectively. These skills provide architectural guidance, generate idiomatic Angular code, and help scaffold new projects using modern best practices.

## Available Skills

- **`angular-developer`**: Generates Angular code and provides architectural guidance. Useful for creating components, services, HTTP communication, or obtaining best practices on reactivity (signals, linkedSignal, resource, httpResource), forms, dependency injection, routing, SSR, accessibility (ARIA), animations, styling, testing, or CLI tooling.
- **`angular-new-app`**: Creates a new Angular app using the Angular CLI. Provides important guidelines for effectively setting up and structuring a modern Angular application.

```bash
npx skills add https://github.com/ofreivogel/claude-angular
```

## Contributions

We welcome contributions to the Angular agent skills. If you would like to contribute to the skills, please make the updates directly in `angular/angular` repository, and to that repository will be output here as a part of our infrastructure setup.

### Feedback & Issues

If you encounter a bug, have feedback, or want to suggest an improvement to the skills, please file an issue in the [angular/angular](https://github.com/angular/angular/issues/new?template=3-docs-bug.yaml) issue tracker. Providing detailed context will help us address your feedback effectively.

### Features & Changes (Pull Requests)

We also accept pull requests for new features, updates, or bug fixes for the skills:

1. Make your changes within the `skills/dev-skills/` directory.
2. Follow the standard Angular [Commit Guidelines](https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md) and [Coding Standards](https://github.com/angular/angular/blob/main/contributing-docs/coding-standards.md).
3. Submit a Pull Request to the main `angular/angular` repository.

<!-- BEGIN DOWNSTREAM: claude-code-plugin -->
<!--
  Everything below this marker is a downstream addition specific to this fork
  and is NOT part of the upstream angular/angular skills export. When syncing
  from upstream, preserve this entire block; conflicts (if any) appear only here.
-->

## Use as a Claude Code plugin

This fork is packaged as two [Claude Code](https://docs.claude.com/en/docs/claude-code) plugins,
served from one marketplace. Both expose the same two skills; they differ only in which CLI they
assume.

| Plugin | For | CLI | MCP server |
| :-- | :-- | :-- | :-- |
| `angular` | Angular CLI workspaces | `ng` | Angular CLI (`ng mcp`) |
| `angular-nx` | Nx workspaces (`nx.json` at the root) | `nx` | none — see below |

**Install one of them, not both.** The skills carry the same names in both plugins
(`angular:angular-developer` vs `angular-nx:angular-developer`), so explicit invocation is
unambiguous — but with both installed, automatic skill selection has two near-identical
candidates to choose between.

```bash
/plugin marketplace add https://github.com/ofreivogel/claude-angular

# Angular CLI workspaces
/plugin install angular@olivers-angular-marketplace

# Nx workspaces
/plugin install angular-nx@olivers-angular-marketplace
```

### Prerequisites

The `angular` plugin declares the `angular-cli` MCP server, which needs the Angular CLI on your
`PATH`:

```bash
npm install -g @angular/cli
ng version
```

The `angular-nx` plugin declares no MCP server on purpose. In an Nx workspace the Angular CLI is
usually not installed, and Nx has its own server covering workspace topics:

```bash
npx nx configure-ai-agents     # sets up the Nx MCP server and the official Nx skills
```

## How the Nx variant is maintained

`nx/` is a translation of the upstream skills, kept as ordinary reviewable repository content —
not a build artifact.

| Path | Role |
| :-- | :-- |
| `angular-developer/`, `angular-new-app/` | Upstream export, never edited |
| `nx/` | The Nx fork: same files, Angular CLI translated to Nx |
| `nx-rules.md` | How the translation is done, with a documentation source per rule |
| `.claude/skills/nx-sync/` | The `/nx-sync` skill that carries out the sync |

Updating after an upstream change:

```bash
git fetch upstream && git merge upstream/22.1.x
# then run /nx-sync, and review with:
git diff nx/
```

`/nx-sync` checks the rules in `nx-rules.md` against the current Nx documentation before applying
them, so commands that Nx has renamed or removed surface as a rule correction instead of as
broken output.

The Nx variant deliberately covers only Angular-specific tooling. Generic Nx topics — project
graph, `affected`, caching, library architecture, module boundaries — are left to the Nx MCP
server and the official Nx skills.

<!-- END DOWNSTREAM: claude-code-plugin -->
