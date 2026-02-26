# Query Use Case Template

Query use cases handle read operations. They are read-only and must not produce side effects. Unlike commands, queries do not create, update, or delete — they only retrieve data.

## File Location

```
<module-path>/src/lib/application/queries/{{entityName}}/
├── get-{{entityName}}s.query.ts
├── get-{{entityName}}.query.ts
└── index.ts
```

---

## Optional Base Class Pattern

You can define an abstract base class in your core library to enforce the query contract:

```typescript
// Define in your core library:
export abstract class QueryBase<TRequest, TResponse> {
  abstract execute(request: TRequest): Promise<Result<TResponse>>;
}
```

Query use cases can then extend this base:

```typescript
export class Get{{EntityName}}sUseCase extends QueryBase<{{EntityName}}FilterParams | void, {{EntityName}}[]> {
  // ...
}
```

This is optional — a plain class with an `execute()` method works equally well. Choose whichever fits your project's conventions.

---

## Template

```typescript
import { inject, Injectable } from '@angular/core';
import { Result } from '@your-org/core';
// Optional: import QueryBase if you use the base class pattern
// import { QueryBase } from '@your-org/core';
import {
  {{EntityName}},
  {{EntityName}}FilterParams,
  {{EntityName}}Id,
  {{EntityName}}RepositoryToken,
} from '../../../domain';

// ============================================
// Get List Query
// ============================================

/**
 * Query use case to get a list of {{entityName}}s.
 *
 * Queries are read-only operations that do not modify state.
 * Accepts optional filter parameters to narrow results.
 *
 * @example
 * ```typescript
 * const useCase = inject(Get{{EntityName}}sUseCase);
 *
 * // Get all
 * const result = await useCase.execute();
 *
 * // Get with filters
 * const filtered = await useCase.execute({ isActive: true });
 *
 * if (result.isSuccess) {
 *   console.log(result.value); // {{EntityName}}[]
 * }
 * ```
 */
@Injectable()
export class Get{{EntityName}}sUseCase {
  private readonly repository = inject({{EntityName}}RepositoryToken);

  /**
   * Execute the query to fetch {{entityName}}s.
   *
   * @param params - Optional filter parameters
   * @returns Result containing array of {{entityName}}s
   */
  async execute(params?: {{EntityName}}FilterParams): Promise<Result<{{EntityName}}[]>> {
    // Repository.findAll() returns Result<{{EntityName}}[]>
    return this.repository.findAll(params);
  }
}

// ============================================
// Get Single Query
// ============================================

/**
 * Query use case to get a single {{entityName}} by ID.
 *
 * @example
 * ```typescript
 * const useCase = inject(Get{{EntityName}}UseCase);
 * const result = await useCase.execute({{EntityName}}Id(1));
 *
 * if (result.isSuccess) {
 *   console.log(result.value); // {{EntityName}}
 * }
 * ```
 */
@Injectable()
export class Get{{EntityName}}UseCase {
  private readonly repository = inject({{EntityName}}RepositoryToken);

  /**
   * Execute the query to fetch a single {{entityName}}.
   *
   * @param id - The {{entityName}} ID (branded type)
   * @returns Result containing the {{entityName}} or error
   */
  async execute(id: {{EntityName}}Id): Promise<Result<{{EntityName}}>> {
    return this.repository.findById(id);
  }
}
```

---

## Barrel Export

Create `<module-path>/src/lib/application/queries/{{entityName}}/index.ts`:

```typescript
export * from './get-{{entityName}}s.query';
export * from './get-{{entityName}}.query';
```

Update `<module-path>/src/lib/application/queries/index.ts`:

```typescript
export * from './{{entityName}}';
```

---

## Registering Query Use Cases as Providers

Add query use cases to the providers in the Infrastructure layer's providers file:

```typescript
export const {{entityName}}Providers: Provider[] = [
  // Repository token binding
  { provide: {{EntityName}}RepositoryToken, useClass: {{EntityName}}RepositoryImpl },
  // Command use cases
  Create{{EntityName}}UseCase,
  Update{{EntityName}}UseCase,
  Delete{{EntityName}}UseCase,
  // Query use cases
  Get{{EntityName}}sUseCase,
  Get{{EntityName}}UseCase,
];
```

---

## Key Principles

1. **`@Injectable()`** — Required for Angular DI
2. **`inject()`** — Function-based DI, never constructor injection
3. **Repository via `InjectionToken`** — Never import the concrete implementation
4. **`Result<T>`** — Always return Result
5. **Read-only** — No side effects; never call `create`, `update`, or `delete` from a query
6. **File suffix** — `.query.ts`
7. **Class suffix** — `UseCase` (consistent with command naming) or `Query` — choose one convention and apply it project-wide

## Naming Conventions

| Type   | File Name                        | Class Name                |
|--------|----------------------------------|---------------------------|
| List   | `get-{{entityName}}s.query.ts`   | `Get{{EntityName}}sUseCase` |
| Single | `get-{{entityName}}.query.ts`    | `Get{{EntityName}}UseCase`  |

> Note: Some teams use the `Query` suffix (e.g., `Get{{EntityName}}sQuery`) instead of `UseCase`. Both are valid — pick one convention and apply it consistently across your project.
