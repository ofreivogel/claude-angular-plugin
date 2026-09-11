# End-to-End (E2E) Testing

> [!IMPORTANT]
> Only use the setup guidelines in this file if there is no existing E2E testing framework configured in the workspace, or if the user has explicitly requested to change or set up E2E testing.

## E2E is a separate project

The key structural difference from an Angular CLI workspace: in Nx, end-to-end tests are their
own project, generated alongside the application. An app at `apps/my-app` gets an e2e project at
`apps/my-app-e2e`, with its own configuration and its own `e2e` target.

This is why e2e is chosen when the application is generated rather than added afterwards.

## Setting Up

The runner is selected with `--e2eTestRunner` when generating the application:

```bash
nx g @nx/angular:application apps/my-app --e2eTestRunner=playwright
```

Accepted values are `playwright` (the generator default), `cypress` and `none`.

`@nx/cypress` is deprecated as of Nx 23 — prefer Playwright for new setups.

To add e2e to an application that was generated with `--e2eTestRunner=none`, install the plugin
and run its configuration generator:

```bash
nx add @nx/playwright
nx g @nx/playwright:configuration --project=my-app-e2e
```

## Running E2E Tests

```bash
nx e2e my-app-e2e
nx e2e my-app-e2e --configuration=production
nx run-many -t e2e
nx affected -t e2e
```

The project name is required.

## Custom & Enterprise Testing Tools

For custom enterprise runners (e.g., Katalon Studio, TestCafe, Selenium), define the execution
command as a target in the e2e project's configuration rather than only as a `package.json`
script — that way it participates in `nx run-many` and `nx affected` like any other target.
