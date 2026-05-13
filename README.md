# Angular Skills

The Angular skills are designed to help coding agents create applications aligned with the latest versions of Angular, best practices, new features and manage Angular applications effectively. These skills provide architectural guidance, generate idiomatic Angular code, and help scaffold new projects using modern best practices.

## Available Skills

- **`angular-developer`**: Generates Angular code and provides architectural guidance. Useful for creating components, services, or obtaining best practices on reactivity (signals, linkedSignal, resource), forms, dependency injection, routing, SSR, accessibility (ARIA), animations, styling, testing, or CLI tooling.
- **`angular-new-app`**: Creates a new Angular app using the Angular CLI. Provides important guidelines for effectively setting up and structuring a modern Angular application.

## Using Agent Skills

Agent Skills are designed to be used with agentic coding tools like [Gemini CLI](https://geminicli.com/docs/cli/skills/), [Antigravity](https://antigravity.google/docs/skills) and more. Activating a skill loads the specific instructions and resources needed for that task.

To use these skills in your own environment you may follow the instructions for your specific tool or use a community tool like [skills.sh](https://skills.sh/).

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
  and is NOT part of the upstream angular/angular skills export. It documents
  how to consume this repo as a Claude Code plugin. When syncing from upstream,
  preserve this entire block; conflicts (if any) appear only here.
-->

## Use as a Claude Code plugin

This fork is additionally packaged as a [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin. It wraps the upstream skills together with a subagent and the Angular CLI MCP server (`ng mcp`). The wrapper lives in `.claude-plugin/`, `.mcp.json`, and `agents/` — these are added on top of the upstream content.

### What the plugin provides

- **Skills**: `angular-developer`, `angular-new-app` (from upstream, namespaced as `angular:angular-developer` etc.)
- **Subagent**: `angular-developer` (Sonnet, preloads both skills, invokes `ng build` after generation)
- **MCP server**: `angular-cli` via `ng mcp` (requires the Angular CLI on `PATH`)

### Install

In a Claude Code session, add this repository as a plugin marketplace and install the plugin:

```bash
/plugin marketplace add https://github.com/ofreivogel/claude-angular
/plugin install angular@angular-skills
```

For local development, point the marketplace at a checkout instead:

```bash
/plugin marketplace add /path/to/claude-angular
/plugin install angular@angular-skills
```

### Requirements

- Angular CLI on `PATH` (`ng version` must succeed) for the MCP server to start.
- Claude Code v2.1.x or newer.

### Verify

```bash
claude plugin validate ./.claude-plugin/plugin.json
claude plugin validate ./.claude-plugin/marketplace.json
```

After install, `claude --debug` shows the plugin load and MCP server initialization.

<!-- END DOWNSTREAM: claude-code-plugin -->
