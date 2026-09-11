# Nx CLI Guide for Agents

In an Nx workspace the Nx CLI (`nx`) manages the workspace. Always prefer Nx commands over
manual file creation or plain `npm` commands when changing project structure or adding
Angular-specific dependencies.

Do not use `ng` here. Nx runs Angular's builders and schematics itself, and the workspace has no
`angular.json` for the Angular CLI to read.

## 1. Managing Dependencies

**Use `nx add` for plugins and Angular libraries** instead of `npm install`. It installs the
package at a version matching your Nx version and runs the package's `init` or `ng-add`
generator, so configuration is wired up for you.

```bash
nx add @nx/angular
nx add @angular/material
nx add tailwindcss
```

To update the workspace and its dependencies, see [migrations.md](migrations.md) — Nx uses a
two-phase `nx migrate` flow, not `ng update`.

## 2. Generating Code (`nx generate` or `nx g`)

Always use the CLI to generate code so it follows workspace conventions and updates the
necessary configuration.

Nx-owned generators take the **path positionally**; the name is derived from the last segment.
There is no `--project` for them.

| Target      | Command                                                        |
| :---------- | :------------------------------------------------------------- |
| Component   | `nx g @nx/angular:component apps/my-app/src/app/foo/foo`       |
| Directive   | `nx g @nx/angular:directive apps/my-app/src/app/foo/foo`       |
| Pipe        | `nx g @nx/angular:pipe apps/my-app/src/app/foo/foo`            |
| Application | `nx g @nx/angular:application apps/my-app`                     |
| Library     | `nx g @nx/angular:library libs/my-lib`                         |

`@nx/angular` provides **no** generator for services, guards, resolvers, interceptors, modules or
environments. For those, pass Angular's own schematic through — and there `--project` is the
correct form:

| Target      | Command                                                          |
| :---------- | :--------------------------------------------------------------- |
| Service     | `nx g @schematics/angular:service my-data --project=my-app`      |
| Guard       | `nx g @schematics/angular:guard auth --project=my-app`           |
| Resolver    | `nx g @schematics/angular:resolver user --project=my-app`        |
| Interceptor | `nx g @schematics/angular:interceptor auth --project=my-app`     |

Never invent an `@nx/angular:<name>`. Run `nx list @nx/angular` to see what actually exists, or
ask the Nx MCP server.

_Note: There is no command to generate a single route definition. Generate a component, then add
it to the `Routes` array manually._

Useful component options: `--inline-style` (`-s`), `--inline-template` (`-t`),
`--change-detection` (`-c`), `--standalone` (default `true`), `--export`.

## 3. Development Server & Proxying

```bash
nx serve my-app
```

The project name is required — `nx serve` on its own fails in a monorepo.

### Backend API Proxying

1. Create `apps/my-app/proxy.conf.json`:
   ```json
   {
     "/api/**": {"target": "http://localhost:3000", "secure": false}
   }
   ```
2. Point the serve target at it in `apps/my-app/project.json`:
   ```json
   "serve": {
     "executor": "@nx/angular:dev-server",
     "options": { "proxyConfig": "apps/my-app/proxy.conf.json" }
   }
   ```

Where targets are inferred by `@nx/angular/plugin` rather than declared, the project may have no
`targets` block at all. Run `nx show project my-app` to see the resolved configuration before
editing.

## 4. Building the Application

```bash
nx build my-app
nx build my-app --configuration=staging
nx run my-app:build:production
```

Modern Angular projects in Nx build with the esbuild-based application executor
(`@nx/angular:application`, a drop-in replacement for Angular's own `application` builder, with
integrated SSR and prerendering). `@nx/angular:browser-esbuild` is the browser-only alternative.

Building several projects at once:

```bash
nx run-many -t build
nx affected -t build
```

## 5. Testing

- **Unit Tests**: `nx test my-app`. See [testing-fundamentals.md](testing-fundamentals.md).
- **End-to-End**: `nx e2e my-app-e2e`. In Nx, e2e is a **separate project**, generated together
  with the application via `--e2eTestRunner=playwright|cypress|none`. See
  [e2e-testing.md](e2e-testing.md).

## 6. Linting

```bash
nx lint my-app
```

## 7. Deployment

Nx has no built-in `deploy` command and `@nx/angular` has no deploy executor. Deployment depends
on the plugin or CI setup in use — add the relevant package and run the target it defines, for
example a `deploy` target declared in the project's configuration. Do not assume a generic
deploy command exists.

## 8. Inspecting the Workspace

```bash
nx show projects              # all projects
nx show project my-app        # resolved configuration, including inferred targets
nx report                     # installed Nx and plugin versions
nx graph                      # project graph
```
