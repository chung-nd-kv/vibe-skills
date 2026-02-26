# Providers Template

The providers array file acts as the composition root for a feature module. It binds domain `InjectionToken`s (ports) to concrete infrastructure implementations, and registers use cases and API clients.

---

## Full Template

```typescript
// File: <module-path>/src/lib/infrastructure/providers.ts

import { Provider } from '@angular/core';

// --- Domain Tokens ---
import { {{EntityName}}RepositoryToken } from '../domain/ports/{{entityName}}-repository.token';

// --- Use Cases ---
import { GetAll{{EntityName}}sUseCase } from '../application/use-cases/get-all-{{entityName}}s.use-case';
import { Get{{EntityName}}ByIdUseCase } from '../application/use-cases/get-{{entityName}}-by-id.use-case';
import { Create{{EntityName}}UseCase } from '../application/use-cases/create-{{entityName}}.use-case';
import { Update{{EntityName}}UseCase } from '../application/use-cases/update-{{entityName}}.use-case';
import { Delete{{EntityName}}UseCase } from '../application/use-cases/delete-{{entityName}}.use-case';

// --- Mappers ---
import { {{EntityName}}Mapper } from '../application/mappers/{{entityName}}.mapper';

// --- API Clients ---
import { {{EntityName}}ApiClient } from './api/{{entityName}}-api.client';

// --- Repository Implementations ---
import { {{EntityName}}RepositoryImpl } from './repositories/{{entityName}}.repository';

export const {{MODULE_NAME}}_PROVIDERS: Provider[] = [
  // Use Cases
  GetAll{{EntityName}}sUseCase,
  Get{{EntityName}}ByIdUseCase,
  Create{{EntityName}}UseCase,
  Update{{EntityName}}UseCase,
  Delete{{EntityName}}UseCase,

  // Mappers
  {{EntityName}}Mapper,

  // API Clients
  {{EntityName}}ApiClient,

  // Repository Bindings (Token → Implementation)
  { provide: {{EntityName}}RepositoryToken, useClass: {{EntityName}}RepositoryImpl },
];
```

---

## InjectionToken Pattern (Domain Port)

The token lives in the domain layer:

```typescript
// File: <module-path>/src/lib/domain/ports/{{entityName}}-repository.token.ts

import { InjectionToken } from '@angular/core';
import { {{EntityName}}Repository } from './{{entityName}}-repository.interface';

export const {{EntityName}}RepositoryToken = new InjectionToken<{{EntityName}}Repository>(
  '{{EntityName}}Repository'
);
```

---

## Route-Level Usage

Register the providers at the route or lazy-loaded module level, not globally:

```typescript
// app.routes.ts or feature routes file
import { Routes } from '@angular/router';
import { {{MODULE_NAME}}_PROVIDERS } from './<module-path>/infrastructure/providers';
import { API_BASE_URL } from './<module-path>/infrastructure/api/{{entityName}}-api.client';
import { environment } from '../environments/environment';

export const routes: Routes = [
  {
    path: '{{entityName}}s',
    providers: [
      { provide: API_BASE_URL, useValue: environment.apiBaseUrl },
      ...{{MODULE_NAME}}_PROVIDERS,
    ],
    loadComponent: () =>
      import('./<module-path>/presentation/{{entityName}}-list/{{entityName}}-list.component').then(
        m => m.{{EntityName}}ListComponent
      ),
  },
];
```

---

## Provider Sections Explained

| Section | Contents | Purpose |
|---|---|---|
| Use Cases | Application use case classes | Injectable business operation handlers |
| Mappers | Mapper classes | Transform DTOs to domain entities and back |
| API Clients | HTTP wrapper classes | Centralized HTTP access per entity |
| Repository Bindings | `{ provide: Token, useClass: Impl }` | Dependency inversion — domain talks to token, infrastructure fulfills it |

---

## Template Variables

| Variable | Description | Example |
|---|---|---|
| `{{EntityName}}` | PascalCase entity name | `Product`, `Order`, `Customer` |
| `{{entityName}}` | camelCase entity name | `product`, `order`, `customer` |
| `{{ModuleName}}` | Screaming snake case module name | `PRODUCT`, `ORDER_MANAGEMENT` |

---

## Notes

- Avoid adding providers with `providedIn: 'root'` inside this array. Route-level providers are preferred for feature isolation and testability.
- Each provider in the array is instantiated once per route scope. If two routes share the same providers array, they each get independent instances.
- To share state between routes, lift the providers to a common parent route.
- API_BASE_URL is intentionally provided separately at the route level so the infrastructure layer remains environment-agnostic.
