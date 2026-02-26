---
name: angular-hexagonal-init
description: Scaffolds Nx Angular libraries with hexagonal architecture folder structure. Use when initializing new feature modules, creating Nx libraries, or setting up domain, application, infrastructure, and presentation layers.
---

# Angular Hexagonal Module Initializer

Scaffolds a complete Nx Angular library with hexagonal architecture (Ports and Adapters) layer structure.

## Inputs Required

| Input | Format | Example |
|-------|--------|---------|
| Module name | kebab-case | `product-catalog` |
| Module title (optional) | Human-readable string | `Product Catalog` |

If Module title is not provided, derive it by converting the kebab-case name to title case (e.g., `product-catalog` → `Product Catalog`).

## Variable Transformations

| Variable | Format | Example |
|----------|--------|---------|
| `{{module-name}}` | kebab-case | `product-catalog` |
| `{{ModuleName}}` | PascalCase | `ProductCatalog` |
| `{{MODULE_NAME}}` | UPPER_SNAKE_CASE | `PRODUCT_CATALOG` |
| `{{Module Title}}` | Human-readable title | `Product Catalog` |

Apply these transformations consistently across all generated files.

---

## Step 1: Validate Input

Before generating anything, validate the module name:

1. Confirm the name matches the pattern `^[a-z][a-z0-9]*(-[a-z0-9]+)*$` (lowercase letters, digits, hyphens — no leading/trailing hyphens, no consecutive hyphens).
2. Check that the library does not already exist at `libs/modules/{{module-name}}/`. If it does, stop and warn the user.
3. If the name contains uppercase letters or underscores, suggest the kebab-case equivalent and ask the user to confirm before proceeding.

---

## Step 2: Generate Nx Library

Run the Nx Angular library generator to create the base library scaffold:

```bash
nx g @nx/angular:library {{module-name}} \
  --directory=libs/modules/{{module-name}} \
  --standalone \
  --prefix=app \
  --tags="scope:modules,type:feature" \
  --no-interactive
```

Flag explanations:

| Flag | Purpose |
|------|---------|
| `--directory=libs/modules/{{module-name}}` | Places the library inside the `libs/modules/` workspace folder |
| `--standalone` | Generates standalone Angular components (no NgModule wrapper) |
| `--prefix=app` | Sets the component selector prefix — change to your org prefix if needed (e.g., `--prefix=my-org`) |
| `--tags="scope:modules,type:feature"` | Nx project tags used for lint boundary enforcement; adjust tag values to match your tagging strategy |
| `--no-interactive` | Runs in non-interactive mode, suitable for scripting |

After generation, verify that `libs/modules/{{module-name}}/src/index.ts` exists before continuing.

---

## Step 3: Create Hexagonal Layer Directories

Create the hexagonal layer folder structure inside the library source:

```bash
# Domain layer
mkdir -p libs/modules/{{module-name}}/src/lib/domain

# Application layer (use cases, queries, mappers, DTOs)
mkdir -p libs/modules/{{module-name}}/src/lib/application/dtos
mkdir -p libs/modules/{{module-name}}/src/lib/application/mappers
mkdir -p libs/modules/{{module-name}}/src/lib/application/queries
mkdir -p libs/modules/{{module-name}}/src/lib/application/usecases

# Infrastructure layer
mkdir -p libs/modules/{{module-name}}/src/lib/infrastructure

# Presentation layer
mkdir -p libs/modules/{{module-name}}/src/lib/presentation
```

This results in the following structure under `src/lib/`:

```
domain/
application/
  dtos/
  mappers/
  queries/
  usecases/
infrastructure/
presentation/
```

---

## Step 4: Generate Boilerplate Index Files

Create a barrel `index.ts` for each layer so that internal imports stay clean and the public API is explicit.

Files to create:

- `src/lib/domain/index.ts`
- `src/lib/application/index.ts`
- `src/lib/application/dtos/index.ts`
- `src/lib/application/mappers/index.ts`
- `src/lib/application/queries/index.ts`
- `src/lib/application/usecases/index.ts`
- `src/lib/infrastructure/index.ts`
- `src/lib/presentation/index.ts`

Each file starts as an empty barrel export (see [references/module-scaffold.md](references/module-scaffold.md) for the exact boilerplate content).

Also update the root `src/index.ts` to re-export the presentation layer as the module's public API:

```typescript
// src/index.ts
export * from './lib/presentation';
```

---

## Step 5: Generate Main Component

Generate the primary feature component inside the presentation layer:

```bash
nx g @angular/core:component {{module-name}} \
  --project={{module-name}} \
  --path=libs/modules/{{module-name}}/src/lib/presentation \
  --standalone \
  --selector=app-{{module-name}} \
  --no-interactive
```

This creates:

```
presentation/
  {{module-name}}/
    {{module-name}}.component.ts
    {{module-name}}.component.html
    {{module-name}}.component.scss
    {{module-name}}.component.spec.ts
```

Export the component from `presentation/index.ts`:

```typescript
export * from './{{module-name}}/{{module-name}}.component';
```

---

## Step 6: Configure Path Alias

After library generation, Nx automatically adds a path alias to `tsconfig.base.json`. Verify that the entry exists and matches the expected import path:

```json
{
  "compilerOptions": {
    "paths": {
      "@your-org/modules-{{module-name}}": [
        "libs/modules/{{module-name}}/src/index.ts"
      ]
    }
  }
}
```

Replace `@your-org` with your workspace's organization scope (e.g., `@acme`, `@myapp`). This prefix is set in `nx.json` or the root `package.json` name field.

Other modules and apps import the feature using this alias:

```typescript
import { {{ModuleName}}Component } from '@your-org/modules-{{module-name}}';
```

Do not use relative paths across library boundaries.

---

## Architecture Overview

### Dependency Rule

Dependencies must always point inward toward the domain:

```
Presentation  ──►  Application  ──►  Domain
Infrastructure ──►  Application  ──►  Domain
```

**Forbidden directions:**

```
Domain        ──X──►  Infrastructure   (domain must not know about adapters)
Application   ──X──►  Presentation     (application must not know about UI)
```

### Layer Responsibilities at a Glance

```
┌─────────────────────────────────────────────────────┐
│  Presentation                                       │
│  Components, routes, route-level providers, pipes   │
│  Depends on: Application                            │
├─────────────────────────────────────────────────────┤
│  Application                                        │
│  Use cases, queries, DTOs, mappers, ports (TS       │
│  interfaces for services)                           │
│  Depends on: Domain                                 │
├─────────────────────────────────────────────────────┤
│  Infrastructure                                     │
│  HTTP adapters, local-storage adapters, signal      │
│  stores, concrete service implementations           │
│  Depends on: Application (implements its ports)     │
├─────────────────────────────────────────────────────┤
│  Domain                                             │
│  Entities, value objects, domain events, domain     │
│  errors, repository/service port interfaces         │
│  Depends on: nothing                                │
└─────────────────────────────────────────────────────┘
```

---

## What Belongs in Each Layer

**Domain**
- Entities and value objects (plain TypeScript classes/interfaces, no Angular)
- Domain error types
- Port interfaces (e.g., `IProductRepository`, `IProductService`)
- Domain events

**Application**
- Use cases (orchestration logic — calls domain ports, not infrastructure directly)
- Query objects and query handlers
- DTOs (input/output shapes crossing the application boundary)
- Mappers (domain entity ↔ DTO conversion)

**Infrastructure**
- Implementations of domain port interfaces
- HTTP services (`HttpClient` usage lives here)
- NgRx / Angular Signals state stores
- Local storage, session storage adapters
- Route-level `providers` arrays (wires ports to implementations)

**Presentation**
- Angular components, directives, pipes
- Component-level state (no business logic)
- Route definitions (`routes.ts`)
- Imports infrastructure providers via route-level injection

---

## Verification Checklist

- [ ] Module name passes kebab-case validation
- [ ] `nx g @nx/angular:library` completed without errors
- [ ] All four layer directories exist: `domain/`, `application/`, `infrastructure/`, `presentation/`
- [ ] Subdirectories created: `application/dtos/`, `application/mappers/`, `application/queries/`, `application/usecases/`
- [ ] Each layer has an `index.ts` barrel file
- [ ] Root `src/index.ts` re-exports from `presentation/`
- [ ] Main component generated in `presentation/{{module-name}}/`
- [ ] Component exported from `presentation/index.ts`
- [ ] Path alias `@your-org/modules-{{module-name}}` exists in `tsconfig.base.json`
- [ ] Nx project tags are set to `scope:modules,type:feature` (or your equivalent)
- [ ] No cross-layer dependency violations (run `nx lint` to confirm)

---

## Related Skills

- **angular-domain-modeling** — Modeling entities, value objects, and port interfaces in the domain layer
- **angular-application-layer** — Writing use cases, queries, DTOs, and mappers in the application layer
- **angular-infrastructure-layer** — Implementing HTTP adapters, signal stores, and provider wiring
- **angular-state-management** — Managing reactive state with NgRx Signals or other state libraries
- **angular-ui-components** — Building and organizing reusable UI components in the presentation layer
