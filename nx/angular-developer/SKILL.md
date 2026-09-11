---
name: angular-developer
description: For Nx workspaces only (an nx.json at the workspace root). Generates Angular code and provides architectural guidance using the Nx CLI. Trigger when creating projects, components, services, or HTTP communication, or for best practices on reactivity (signals, linkedSignal, resource, httpResource), forms, dependency injection, routing, SSR, accessibility (ARIA), animations, styling (component styles, Tailwind CSS), testing, naming conventions, or Nx tooling.
license: MIT
metadata:
  author: Copyright 2026 Google LLC
  version: '1.0'
---

# Angular Developer Guidelines (Nx)

0. **This skill applies to Nx workspaces only.** Confirm that `nx.json` exists at the workspace root before following it. If it does not, stop and use the Angular CLI version of this skill instead.

1. Always analyze the project's Angular and Nx version before providing guidance, as best practices and available features vary significantly between versions. `nx report` lists the installed versions.

2. When generating code, follow Angular's style guide and best practices for maintainability and performance. Use the Nx CLI for scaffolding components, services, directives, pipes, and routes to ensure consistency. Never use `ng` in an Nx workspace — Nx runs Angular's builders and schematics itself, and there is no `angular.json` for the Angular CLI to read.

3. Once you finish generating code, run the build to ensure there are no build errors: `nx build <project>`, or `nx affected -t build` to catch breakage in dependent projects. If there are errors, analyze the error messages and fix them before proceeding. Do not skip this step, as it is critical for ensuring the generated code is correct and functional.

## Creating Projects

If no guidelines are provided by the user, here are some default rules to follow when creating a new Angular project:

1. Use the latest stable version of Angular unless the user specifies otherwise.
2. Use Signal Forms for form management in new projects (stable in Angular v22 and newer) [Find out more](references/signal-forms.md).

**Inside an existing Nx workspace**, an "Angular project" is an application or library in that
workspace, not a new workspace:

```bash
nx g @nx/angular:application apps/<name>
nx g @nx/angular:library libs/<name>
```

**Creating a new workspace** is a separate step: `npx create-nx-workspace@latest <name>
--preset=angular-monorepo`. Do not use the Angular CLI to scaffold a workspace that is meant to
be an Nx workspace.

## Generating Angular Code

How to run tasks and discover generators is Nx's own territory — use the Nx MCP server and the
Nx skills for that. What is specific to Angular, and easy to get wrong, is **which** generator
to reach for:

**Nx-owned generators take the path positionally.** The name is derived from the last segment,
and there is no `--project`:

```bash
nx g @nx/angular:component apps/my-app/src/app/foo/foo
nx g @nx/angular:directive apps/my-app/src/app/foo/foo
nx g @nx/angular:pipe apps/my-app/src/app/foo/foo
```

**`@nx/angular` has no generator for services, guards, resolvers, interceptors, modules,
environments, classes, interfaces or enums.** Pass Angular's own schematic through — and there
`--project` is the correct form:

```bash
nx g @schematics/angular:service my-data --project=my-app
nx g @schematics/angular:guard auth --project=my-app
nx g @schematics/angular:interceptor auth --project=my-app
nx g @schematics/angular:resolver user --project=my-app
nx g @schematics/angular:environments --project=my-app
```

Never invent an `@nx/angular:<name>`. Run `nx list @nx/angular` to see what exists, or ask the
Nx MCP server.

_Note: There is no command to generate a single route definition. Generate a component, then add
it to the `Routes` array manually._

## Components

When working with Angular components, consult the following references based on the task:

- **Fundamentals**: Anatomy, metadata, core concepts, self-closing tags, and template control flow (@if, @for, @switch). Read [components.md](references/components.md)
- **Inputs**: Signal-based inputs, transforms, and model inputs. Read [inputs.md](references/inputs.md)
- **Outputs**: Signal-based outputs and custom event best practices. Read [outputs.md](references/outputs.md)
- **Host Elements**: Host bindings and attribute injection. Read [host-elements.md](references/host-elements.md)
- **Naming Conventions**: Modern Angular v20+ naming style ("Intent over Role") for files, components, services, directives, pipes, and models. Read [naming-conventions.md](references/naming-conventions.md)

If you require deeper documentation not found in the references above, read the documentation at `https://angular.dev/guide/components`.

## Reactivity and Data Management

When managing state and data reactivity, use Angular Signals and consult the following references:

- **Signals Overview**: Core signal concepts (`signal`, `computed`), reactive contexts, and `untracked`. Read [signals-overview.md](references/signals-overview.md)
- **Dependent State (`linkedSignal`)**: Creating writable state linked to source signals. Read [linked-signal.md](references/linked-signal.md)
- **Async Reactivity (`resource`)**: Fetching asynchronous data directly into signal state. Read [resource.md](references/resource.md)
- **Side Effects (`effect`)**: Logging, third-party DOM manipulation (`afterRenderEffect`), and when NOT to use effects. Read [effects.md](references/effects.md)

## HTTP Communication

When communicating with backend services, use Angular HTTP APIs and consult the following reference:

- **HTTP Client and Resources**: `provideHttpClient`, `HttpClient`, interceptors, and `httpResource`. Read [http-client.md](references/http-client.md)

## Forms

In most cases for new apps, **prefer signal forms**. When making a forms decision, analyze the project and consider the following guidelines:

- If the application is using v22 or newer and this is a new form, **prefer Signal Forms**.
- For older applications or when working with existing forms, use the appropriate form type that matches the applications current form strategy.

- **Signal Forms**: Use signals for form state management. Read [signal-forms.md](references/signal-forms.md)
- **Template-driven forms**: Use for simple forms. Read [template-driven-forms.md](references/template-driven-forms.md)
- **Reactive forms**: Use for complex forms. Read [reactive-forms.md](references/reactive-forms.md)

## Dependency Injection

When implementing dependency injection in Angular, follow these guidelines:

- **Fundamentals**: Overview of Dependency Injection, services, and the `inject()` function. Read [di-fundamentals.md](references/di-fundamentals.md)
- **Creating and Using Services**: Creating services, the `providedIn: 'root'` option, and injecting into components or other services. Read [creating-services.md](references/creating-services.md)
- **Defining Dependency Providers**: Automatic vs manual provision, `InjectionToken`, `useClass`, `useValue`, `useFactory`, and scopes. Read [defining-providers.md](references/defining-providers.md)
- **Injection Context**: Where `inject()` is allowed, `runInInjectionContext`, and `assertInInjectionContext`. Read [injection-context.md](references/injection-context.md)
- **Hierarchical Injectors**: The `EnvironmentInjector` vs `ElementInjector`, resolution rules, modifiers (`optional`, `skipSelf`), and `providers` vs `viewProviders`. Read [hierarchical-injectors.md](references/hierarchical-injectors.md)

## Pipes

When formatting values in templates, creating custom pipes, or reusing pipe-like logic in TypeScript, consult the following reference. Prefer pipes in templates; outside templates, avoid injecting pipe classes just to call `transform()`.

- **Pipes**: Built-in pipe imports, custom pipe naming and implementation, pure vs impure pipes, and TypeScript reuse patterns using standalone formatting functions or extracted plain functions. Read [pipes.md](references/pipes.md)

## Angular Aria

When building accessible custom components for any of the following patterns: Accordion, Listbox, Combobox, Menu, Tabs, Toolbar, Tree, Grid, consult the following reference:

- **Angular Aria Components**: Building headless, accessible components (Accordion, Listbox, Combobox, Menu, Tabs, Toolbar, Tree, Grid) and styling ARIA attributes. Read [angular-aria.md](references/angular-aria.md)

## Routing

When implementing navigation in Angular, consult the following references:

- **Define Routes**: URL paths, static vs dynamic segments, wildcards, and redirects. Read [define-routes.md](references/define-routes.md)
- **Route Loading Strategies**: Eager vs lazy loading, and context-aware loading. Read [loading-strategies.md](references/loading-strategies.md)
- **Show Routes with Outlets**: Using `<router-outlet>`, nested outlets, and named outlets. Read [show-routes-with-outlets.md](references/show-routes-with-outlets.md)
- **Navigate to Routes**: Declarative navigation with `RouterLink` and programmatic navigation with `Router`. Read [navigate-to-routes.md](references/navigate-to-routes.md)
- **Control Route Access with Guards**: Implementing `CanActivate`, `CanMatch`, and other guards for security. Read [route-guards.md](references/route-guards.md)
- **Data Resolvers**: Pre-fetching data before route activation with `ResolveFn`. Read [data-resolvers.md](references/data-resolvers.md)
- **Router Lifecycle and Events**: Chronological order of navigation events and debugging. Read [router-lifecycle.md](references/router-lifecycle.md)
- **Rendering Strategies**: CSR, SSG (Prerendering), and SSR with hydration. Read [rendering-strategies.md](references/rendering-strategies.md)
- **Route Transition Animations**: Enabling and customizing the View Transitions API. Read [route-animations.md](references/route-animations.md)

If you require deeper documentation or more context, visit the [official Angular Routing guide](https://angular.dev/guide/routing).

## Styling and Animations

When implementing styling and animations in Angular, consult the following references:

- **Using Tailwind CSS with Angular**: Integrating Tailwind CSS into Angular projects. Read [tailwind-css.md](references/tailwind-css.md)
- **Angular Animations**: Using native CSS (recommended) or the legacy DSL for dynamic effects. Read [angular-animations.md](references/angular-animations.md)
- **Styling components**: Best practices for component styles and encapsulation. Read [component-styling.md](references/component-styling.md)

## Testing

When writing or updating tests, consult the following references based on the task:

- **Fundamentals**: Best practices for unit testing (Vitest), async patterns, and `TestBed`. Read [testing-fundamentals.md](references/testing-fundamentals.md)
- **Component Harnesses**: Standard patterns for robust component interaction. Read [component-harnesses.md](references/component-harnesses.md)
- **Router Testing**: Using `RouterTestingHarness` for reliable navigation tests. Read [router-testing.md](references/router-testing.md)
- **End-to-End (E2E) Testing**: Setting up and running E2E tests. Read [e2e-testing.md](references/e2e-testing.md)

## Tooling

When working with Angular tooling, consult the following references:

- **Version Updates and Code Modernization**: The two-phase `nx migrate` flow and Angular's refactoring schematics. Read [migrations.md](references/migrations.md)
- **Environment Configuration**: Strategies for build-time and runtime configuration. Read [environment-configuration.md](references/environment-configuration.md)
