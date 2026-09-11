# Nx MCP Server

Nx ships a Model Context Protocol (MCP) server that lets AI assistants interact with the
workspace: project graph, project-to-file mapping, runnable targets, available generators, and
Nx documentation. In an Nx workspace it replaces the MCP server bundled with the Angular CLI:
that CLI is usually not installed here, and its workspace tools read `angular.json`, which does
not exist under Nx (per-project `project.json` plus `nx.json` instead).

## Setup

The supported way to configure it, together with the agent config files and the official Nx
skills, is:

```bash
npx nx configure-ai-agents
```

Select `claude` when prompted. For Claude Code this installs the Nx skills as a plugin rather
than copying files into the workspace.

To register the server directly instead:

```bash
claude mcp add nx-mcp npx nx mcp
```

## Command

- Nx >= 21.4: `npx nx mcp`
- Older versions: `npx nx-mcp@latest`

Options include `--transport` (`stdio`, `sse`, `http`) and `--port` for the HTTP/SSE port
(default `9921`).

Manual configuration, for hosts that read an MCP config file:

```json
{
  "servers": {
    "nx-mcp": {
      "type": "stdio",
      "command": "npx",
      "args": ["nx", "mcp"]
    }
  }
}
```

## What it provides

- **Workspace understanding** — project relationships through the project graph, so cross-project
  impact can be reasoned about instead of guessed.
- **Task discovery** — which targets a project actually has, including targets inferred by
  plugins such as `@nx/angular/plugin` that appear in no `project.json`.
- **Generator discovery** — which generators are installed and what options they take, which is
  what keeps generated code consistent with the workspace conventions.
- **Nx documentation** — answers grounded in the current docs rather than in memorised commands.

## Angular-specific guidance

The Nx MCP server covers workspace and tooling questions. For Angular framework guidance
(signals, forms, dependency injection, routing, accessibility), use the references in this
skill.
