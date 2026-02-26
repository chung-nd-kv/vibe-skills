# Module Scaffold Reference

This file documents every file and directory generated for a hexagonal module named `{{module-name}}` (PascalCase: `{{ModuleName}}`, UPPER_SNAKE_CASE: `{{MODULE_NAME}}`).

---

## Complete Directory Tree

```
libs/modules/{{module-name}}/
├── project.json
├── tsconfig.json
├── tsconfig.lib.json
├── tsconfig.spec.json
├── jest.config.ts
└── src/
    ├── index.ts                                          ← public API (re-exports presentation)
    └── lib/
        ├── domain/
        │   └── index.ts                                  ← domain barrel
        ├── application/
        │   ├── index.ts                                  ← application barrel
        │   ├── dtos/
        │   │   └── index.ts                              ← DTOs barrel
        │   ├── mappers/
        │   │   └── index.ts                              ← mappers barrel
        │   ├── queries/
        │   │   └── index.ts                              ← queries barrel
        │   └── usecases/
        │       └── index.ts                              ← use cases barrel
        ├── infrastructure/
        │   ├── index.ts                                  ← infrastructure barrel
        │   └── {{module-name}}.providers.ts              ← route-level DI providers
        └── presentation/
            ├── index.ts                                  ← presentation barrel
            └── {{module-name}}/
                ├── {{module-name}}.component.ts
                ├── {{module-name}}.component.html
                ├── {{module-name}}.component.scss
                ├── {{module-name}}.component.spec.ts
                └── routes.ts
```

---

## Boilerplate Index Files

Each layer starts with an empty barrel. Add exports as you add files to that layer.

### `src/index.ts` (public API)

```typescript
// Public API — only expose what consumers need.
// Presentation layer is the entry point for feature consumers.
export * from './lib/presentation';
```

### `src/lib/domain/index.ts`

```typescript
// Domain barrel — export entities, value objects, port interfaces, domain errors.
```

### `src/lib/application/index.ts`

```typescript
// Application barrel — re-exports from sub-folders.
export * from './dtos';
export * from './mappers';
export * from './queries';
export * from './usecases';
```

### `src/lib/application/dtos/index.ts`

```typescript
// DTOs barrel — export all input/output DTO types for {{ModuleName}}.
```

### `src/lib/application/mappers/index.ts`

```typescript
// Mappers barrel — export mapper functions/classes for {{ModuleName}}.
```

### `src/lib/application/queries/index.ts`

```typescript
// Queries barrel — export query objects and query handler tokens.
```

### `src/lib/application/usecases/index.ts`

```typescript
// Use cases barrel — export use case classes and injection tokens.
```

### `src/lib/infrastructure/index.ts`

```typescript
// Infrastructure barrel — export adapter implementations and provider factory.
export * from './{{module-name}}.providers';
```

### `src/lib/presentation/index.ts`

```typescript
// Presentation barrel — export the feature component and routes.
export * from './{{module-name}}/{{module-name}}.component';
export * from './{{module-name}}/routes';
```

---

## Sample `routes.ts`

Routes are defined in the presentation layer. Infrastructure providers are injected at route level so they are lazily loaded with the feature and do not pollute the root injector.

```typescript
// src/lib/presentation/{{module-name}}/routes.ts
import { Route } from '@angular/router';
import { provide{{ModuleName}}Providers } from '../../infrastructure/{{module-name}}.providers';
import { {{ModuleName}}Component } from './{{module-name}}.component';

export const {{MODULE_NAME}}_ROUTES: Route[] = [
  {
    path: '',
    providers: [
      // Wire infrastructure adapters to application ports at route level.
      ...provide{{ModuleName}}Providers(),
    ],
    children: [
      {
        path: '',
        component: {{ModuleName}}Component,
      },
    ],
  },
];
```

Consuming applications lazy-load the module:

```typescript
// In the shell app routes
{
  path: '{{module-name}}',
  loadChildren: () =>
    import('@your-org/modules-{{module-name}}').then(m => m.{{MODULE_NAME}}_ROUTES),
}
```

---

## Infrastructure Providers Barrel Template

The providers file is the only place where application ports are mapped to infrastructure adapters. This keeps infrastructure wiring out of the domain and application layers.

```typescript
// src/lib/infrastructure/{{module-name}}.providers.ts
import { EnvironmentProviders, makeEnvironmentProviders } from '@angular/core';

// Import application-layer port tokens once they are defined, for example:
// import { {{MODULE_NAME}}_REPOSITORY_TOKEN } from '../application/usecases';

// Import infrastructure adapter implementations, for example:
// import { {{ModuleName}}HttpRepository } from './adapters/{{module-name}}-http.repository';

/**
 * Provides all infrastructure adapters for the {{ModuleName}} feature.
 * Call this inside the route-level `providers` array.
 */
export function provide{{ModuleName}}Providers(): EnvironmentProviders {
  return makeEnvironmentProviders([
    // Map ports to adapters here, for example:
    // { provide: {{MODULE_NAME}}_REPOSITORY_TOKEN, useClass: {{ModuleName}}HttpRepository },
  ]);
}
```

---

## Naming Conventions Summary

| Artifact | Pattern | Example |
|----------|---------|---------|
| Library folder | `libs/modules/{{module-name}}/` | `libs/modules/product-catalog/` |
| Component class | `{{ModuleName}}Component` | `ProductCatalogComponent` |
| Component selector | `app-{{module-name}}` | `app-product-catalog` |
| Routes constant | `{{MODULE_NAME}}_ROUTES` | `PRODUCT_CATALOG_ROUTES` |
| Providers function | `provide{{ModuleName}}Providers` | `provideProductCatalogProviders` |
| Injection token | `{{MODULE_NAME}}_<ROLE>_TOKEN` | `PRODUCT_CATALOG_REPOSITORY_TOKEN` |
| Port interface | `I{{ModuleName}}<Role>` | `IProductCatalogRepository` |
| Adapter class | `{{ModuleName}}<Adapter>` | `ProductCatalogHttpRepository` |
| DTO | `{{ModuleName}}<Action>Dto` | `ProductCatalogCreateDto` |
| Mapper | `{{ModuleName}}<Entity>Mapper` | `ProductCatalogItemMapper` |
| Use case class | `{{ModuleName}}<Action>UseCase` | `ProductCatalogFetchUseCase` |
| Query class | `{{ModuleName}}<Description>Query` | `ProductCatalogListQuery` |
