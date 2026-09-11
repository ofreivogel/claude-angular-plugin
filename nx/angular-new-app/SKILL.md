---
name: angular-new-app
description: Creates a new Nx workspace with an Angular application using create-nx-workspace. This skill should be used whenever a user wants to create a new Angular application in an Nx monorepo, and contains important guidelines for how to effectively create a modern Nx Angular workspace.
license: MIT
compatibility: Requires node, npm, and access to the internet
metadata:
  author: Angular Team @ Google, adapted for Nx
  version: '1.0'
---

# Nx Angular New App

You are an expert in TypeScript, Angular, Nx and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular, Nx and TypeScript best practices.

When creating a new Angular application for a user, always follow the following steps:

1. **Decide whether a new workspace is needed.**
   - If the current directory already is an Nx workspace (`nx.json` at the root), do **not** create a new workspace. Add an application to the existing one:

     `npx nx g @nx/angular:application apps/<app-name>`

   - Only if there is no workspace yet, continue to step 2.

2. **Create the workspace**: Suggest a name based on the user prompt or ask for one. Create it with:

   `npx create-nx-workspace@latest <name> --preset=angular-monorepo --interactive=false --aiAgents=claude`

   Available Angular presets:
   - `angular-monorepo` — `apps/` and `libs/` layout, the usual choice
   - `angular-standalone` — a single application at the root, no `apps/` directory
   - `angular` — prompts for one of the two above

   _Important_: Prefer `--aiAgents=claude`, or the option matching the environment (`codex`, `copilot`, `cursor`, `gemini`, `opencode`, `none`). This also sets up the Nx MCP server and the official Nx skills, which cover workspace-level topics this skill deliberately does not.

   Consider these commonly useful flags based on the user's requirements:
   - `--style=scss|css|less` — stylesheet format
   - `--ssr` — enable server-side rendering and prerendering
   - `--bundler=esbuild|rspack|webpack` — defaults to esbuild
   - `--unitTestRunner=vitest-angular|vitest-analog|jest|none`
   - `--e2eTestRunner=playwright|cypress|none` — defaults to Playwright
   - `--appName=<name>` — application name if it differs from the workspace name
   - `--packageManager=npm|yarn|pnpm|bun`
   - `--nxCloud=skip` — unless the user wants Nx Cloud

   After creation, load the generated agent configuration (`CLAUDE.md` / `AGENTS.md`) into memory so that generated code stays consistent with the workspace conventions.

3. **Converting an existing Angular CLI workspace** instead of creating a new one:

   `npx nx@latest init` — keeps the single-application layout
   `npx nx@latest init --integrated` — moves to the full `apps/` + `libs/` monorepo layout

   This installs `nx` and `@nx/workspace`, creates `nx.json`, and splits `angular.json` into one `project.json` per project.

4. Do not start the app until you've built some features; ask the user if they want to start it. You can always run `npx nx build <project>` to check for errors and repair them.

5. Remember the following guidelines for continuing to generate Angular application code.

   Nx-owned generators take the **path positionally** — the name is derived from the last segment, and there is no `--project`:
   - Component: `npx nx g @nx/angular:component apps/<app>/src/app/<name>/<name>`
   - Directive: `npx nx g @nx/angular:directive apps/<app>/src/app/<name>/<name>`
   - Pipe: `npx nx g @nx/angular:pipe apps/<app>/src/app/<name>/<name>`
   - Library: `npx nx g @nx/angular:library libs/<name>`

   `@nx/angular` has **no** generator for services, guards, resolvers, interceptors, modules, interfaces, enums or classes. For those, pass Angular's own schematic through — and there `--project` is the correct form:
   - Service: `npx nx g @schematics/angular:service <name> --project=<app>`
   - Guard: `npx nx g @schematics/angular:guard <name> --project=<app>`
   - Interceptor: `npx nx g @schematics/angular:interceptor <name> --project=<app>`
   - Resolver: `npx nx g @schematics/angular:resolver <name> --project=<app>`
   - Interface / Enum / Class: `npx nx g @schematics/angular:<type> <name> --project=<app>`

   Never invent an `@nx/angular:<name>`. Run `npx nx list @nx/angular` to see what exists, or ask the Nx MCP server.

   _IMPORTANT_: Take note of the path returned from running the generate commands so that you know exactly where the new files are.

   Use the CLI to generate the code, then augment it to meet the needs of the application.

6. To add tailwind, run `npx nx add tailwindcss`. After that, you can start using tailwind classes in your Angular application. Follow the best practices for tailwind v4 here, learn more if needed: https://tailwindcss.com/docs/upgrade-guide.

_IMPORTANT_: The Nx MCP server exposes the project graph, the real target list (including targets inferred by plugins that appear in no `project.json`) and the installed generators with their options. Set it up with `npx nx configure-ai-agents` and prefer it over guessing workspace structure.
