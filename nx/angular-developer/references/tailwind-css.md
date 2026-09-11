# Using Tailwind CSS with Angular in an Nx Workspace

Tailwind CSS is a utility-first CSS framework that integrates seamlessly with Angular.

**CRITICAL AGENT GUIDANCE: ALWAYS focus on Tailwind CSS v4 practices. DO NOT revert to old Tailwind v3 patterns (like creating `tailwind.config.js` with `@tailwind` directives) as this will break the application build. Modern Angular projects use Tailwind v4.**

## Setup

Nx has **no** Tailwind generator — `@nx/angular:setup-tailwind` was removed. Install Tailwind the
way the official [Tailwind CSS with Angular](https://tailwindcss.com/docs/installation/framework-guides/angular)
guide describes, then apply the Nx-specific scanning setup below.

### 1. Install Dependencies

```shell
npm install tailwindcss @tailwindcss/postcss postcss
```

### 2. Configure PostCSS

Create a `.postcssrc.json` in the **project** root (e.g. `apps/my-app/.postcssrc.json`), not the
workspace root:

```json
{
  "plugins": {
    "@tailwindcss/postcss": {}
  }
}
```

_(Do NOT create a `tailwind.config.js` file! Configuration in v4 is handled through CSS variables)._

### 3. Import Tailwind CSS — and restrict what it scans

This is the step that differs in a monorepo, and getting it wrong is expensive.

Tailwind v4 detects classes from source files automatically, but Angular's PostCSS plugin scans
from the **workspace root**. In an Nx workspace that means every library gets scanned, and
production CSS is bloated with utilities no one uses.

Narrow the scan to the application, then declare the libraries it actually depends on:

```css
/* apps/my-app/src/styles.css */
@import 'tailwindcss' source('./app');

@source '../../../libs/ui';
@source '../../../libs/shared';
```

_(If using SCSS, use `@use 'tailwindcss';` instead)._

### 4. Keep the source directives in sync

Maintaining that `@source` list by hand goes stale as dependencies change. `@juristr/nx-tailwind-sync`
derives it from the project graph. Register it on the targets that build CSS:

```json
{
  "targets": {
    "build": {
      "executor": "@angular/build:application",
      "syncGenerators": ["@juristr/nx-tailwind-sync:source-directives"]
    }
  }
}
```

`nx build` and `nx serve` then update `styles.css` with the correct directives.

### 5. Use Utility Classes

```html
<h1 class="text-3xl font-bold underline">Hello world!</h1>
```

## Summary for AI Agents

- **Do not use `@tailwind base; @tailwind components; @tailwind utilities;`**. Use `@import 'tailwindcss';`.
- **Do not create `tailwind.config.js`**. Configuration is managed directly in CSS via theme variables or using PostCSS configurations.
- **Do not look for an Nx Tailwind generator** — there is none. Install manually.
- **Always restrict scanning** with `source()` and `@source` in a monorepo, or prefer
  `@juristr/nx-tailwind-sync` to derive them from the project graph.
- Stick strictly to v4 syntax and workflows.
