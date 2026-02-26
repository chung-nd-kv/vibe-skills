---
name: angular-infrastructure-layer
description: Creates infrastructure layer artifacts for Angular hexagonal architecture including HTTP API clients, repository implementations that implement domain ports, and provider arrays that bind InjectionTokens to implementations.
---

# Angular Infrastructure Layer

Creates infrastructure layer artifacts — the adapters in hexagonal architecture that implement domain ports using external services like HTTP APIs.

## Architecture Position

The infrastructure layer sits at the outermost ring of hexagonal architecture:

```
Domain (entities, value objects, ports)
  ↑
Application (use cases, DTOs, mappers)
  ↑
Infrastructure (API clients, repository implementations)
  ↑
Presentation (components, view models)
```

Dependencies:
- Depends on: Domain layer (ports/interfaces) + Application layer (DTOs, mappers)
- Provides: Concrete implementations for domain port interfaces
- Never imported by: Domain or Application layers

---

## Creating API Clients

API clients are `@Injectable()` classes that wrap `HttpClient` calls and return typed results. They do NOT implement domain ports directly — they are consumed by repository implementations.

### InjectionToken for Base URL

Define a token so the base URL can be provided at route level or app level:

```typescript
import { InjectionToken } from '@angular/core';

export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');

// Provide in app.config.ts or route providers:
// { provide: API_BASE_URL, useValue: environment.apiBaseUrl }
```

### Error Extraction Helper

Use a standalone function (not an imported utility) to normalize API errors:

```typescript
function extractApiError(error: unknown): string {
  if (error instanceof Error) return error.message;
  if (typeof error === 'object' && error !== null && 'message' in error) {
    return String((error as any).message);
  }
  return 'An unexpected error occurred';
}
```

### API Client Rules

- Use `@Injectable()` without `providedIn` (provisioned at route level via providers array)
- Use `inject(HttpClient)` and `inject(API_BASE_URL)` for dependencies
- Use `firstValueFrom` from `rxjs` to convert `Observable` → `Promise`
- Standard methods: `findAll`, `findById`, `create`, `update`, `delete`
- Use a `buildQueryParams` helper for query string construction
- Return plain DTOs (no domain types)

### File Location

```
<module-path>/src/lib/infrastructure/api/<entity-name>-api.client.ts
```

See [`references/api-client-template.md`](references/api-client-template.md) for the full template.

---

## Creating Repository Implementations

Repository implementations implement the domain port interface defined in the domain layer. They coordinate between the API client and a mapper.

### Repository Implementation Rules

- Class must `implement` the domain port interface (e.g., `EntityRepository`)
- Use `inject()` for the API client and mapper
- All methods wrap results in `Result<T>` using try/catch
- On success: `Result.success(mappedValue)`
- On failure: `Result.failure(extractApiError(error))`
- No business logic — only data fetching and mapping

### File Location

```
<module-path>/src/lib/infrastructure/repositories/<entity-name>.repository.ts
```

See [`references/repository-impl-template.md`](references/repository-impl-template.md) for the full template.

---

## Provider Binding Pattern

Providers bind domain `InjectionToken`s (ports) to infrastructure implementations. This is how dependency inversion is achieved at runtime.

### Binding Structure

```typescript
import { Provider } from '@angular/core';
import { EntityRepositoryToken } from '../domain/ports/entity-repository.token';
import { EntityRepositoryImpl } from './repositories/entity.repository';

export const ENTITY_INFRASTRUCTURE_PROVIDERS: Provider[] = [
  { provide: EntityRepositoryToken, useClass: EntityRepositoryImpl },
];
```

### Route-Level Provisioning vs `providedIn: 'root'`

| Approach | When to Use |
|---|---|
| Route-level providers array | Feature modules, lazy-loaded routes, scoped state |
| `providedIn: 'root'` | Truly singleton, app-wide services (rare in hexagonal arch) |

For hexagonal architecture, prefer route-level provisioning so each feature module controls its own bindings.

### Provider Array File Structure

```
<module-path>/src/lib/infrastructure/
  providers.ts          ← exports combined provider array
  api/
    <entity>-api.client.ts
  repositories/
    <entity>.repository.ts
```

See [`references/providers-template.md`](references/providers-template.md) for the full template.

---

## Infrastructure Layer File Layout

```
<module-path>/src/lib/
├── domain/
│   ├── entities/
│   │   └── entity.ts
│   └── ports/
│       └── entity-repository.token.ts    ← InjectionToken<EntityRepository>
├── application/
│   ├── dtos/
│   │   └── entity.dto.ts
│   └── mappers/
│       └── entity.mapper.ts
└── infrastructure/
    ├── api/
    │   └── entity-api.client.ts          ← HttpClient wrapper
    ├── repositories/
    │   └── entity.repository.ts          ← implements domain port
    └── providers.ts                      ← binds tokens to impls
```

---

## Key Principles

1. **No business logic in infrastructure** — infrastructure only fetches, maps, and returns data.
2. **API client wraps HttpClient** — converts Observables to Promises, handles HTTP params, centralizes endpoint paths.
3. **Repositories use API client + Mapper** — they do not call `HttpClient` directly.
4. **All repository methods return `Result<T>`** — callers in the application layer receive a discriminated union, never raw exceptions.
5. **Provider bindings are separate from Token and Implementation** — the `providers.ts` file is the only place that knows both sides.

---

## Verification Checklist

- [ ] API client uses `@Injectable()` without `providedIn`
- [ ] `API_BASE_URL` InjectionToken is defined and provided
- [ ] API client uses `firstValueFrom` (not `.subscribe()`)
- [ ] Repository class implements the domain port interface
- [ ] All repository methods wrap results in `Result<T>` with try/catch
- [ ] No HttpClient calls exist directly inside repository implementations
- [ ] Provider array binds the domain token to the infrastructure implementation
- [ ] Providers are registered at route level (not globally unless intentional)
- [ ] Infrastructure files import from domain/application — not the other way around

---

## Related Skills

- `angular-domain-layer` — creates domain entities, value objects, and port tokens
- `angular-application-layer` — creates use cases, DTOs, and mappers
- `angular-presentation-layer` — creates components and view models that consume use cases
