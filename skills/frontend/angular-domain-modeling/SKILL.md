---
name: angular-domain-modeling
description: Creates domain layer artifacts for Angular hexagonal architecture including entities with branded IDs, value objects, repository port interfaces with InjectionToken, and Result type error handling patterns.
---

# Angular Domain Modeling

Creates domain layer artifacts following hexagonal architecture. The domain layer is the innermost layer with no outward dependencies — pure TypeScript only.

## Architecture Position

```
UI Layer (Angular Components)
      |
Application Layer (Use Cases / Commands / Queries)
      |
Domain Layer  <-- YOU ARE HERE
      |
Infrastructure Layer (implements domain ports)
```

The domain layer:
- Depends on **nothing** — no Angular imports, no HTTP, no RxJS
- Contains entities, value objects, repository port interfaces, and domain events
- Is the most stable layer; changes only when business rules change

## The Result\<T\> Pattern

All factory functions and operations that can fail return `Result<T>` instead of throwing exceptions.

```typescript
export interface Result<T> {
  readonly isSuccess: boolean;
  readonly value?: T;
  readonly errorMessage?: string;
}
export const Result = {
  success: <T>(value: T, message?: string): Result<T> => ({ isSuccess: true, value, errorMessage: message }),
  failure: <T>(error: string): Result<T> => ({ isSuccess: false, errorMessage: error }),
};
```

> Define this in your project's shared core library (e.g., `@your-org/core`). Alternatively, use libraries like `neverthrow`.

## Creating Domain Entities

### Steps

1. Identify the entity name and its core business properties.
2. Create the file at the canonical location.
3. Define a branded ID type to prevent accidental ID mix-ups.
4. Define the readonly interface.
5. Write the factory function with validation that returns `Result<T>`.
6. Add pure helper functions (`equals`, `update`, etc.) as standalone exports — do not put logic inside a class.

### File Location

```
<module-path>/src/lib/domain/entities/<entity-name>.entity.ts
```

### Variable Substitution

| Placeholder | Format | Example |
|---|---|---|
| `EntityName` | PascalCase | `Product` |
| `entityName` | camelCase | `product` |
| `entity-name` | kebab-case | `product` |

### What the File Must Contain

- **Branded ID type** — prevents passing the wrong ID to the wrong function
- **ID constructor** — `createEntityNameId(raw: string): EntityNameId`
- **Readonly interface** — all properties are `readonly`
- **Factory function** — validates inputs, returns `Result<EntityName>`
- **Equality check** — `equalsEntityName(a, b): boolean` compares by ID
- **Pure update function** — `updateEntityName(entity, changes): Result<EntityName>` returns a new object

### Branded ID Example

```typescript
// Branded type prevents passing a ProductId where a CategoryId is expected
export type ProductId = string & { readonly __brand: 'ProductId' };

export const createProductId = (raw: string): ProductId => {
  if (!raw || raw.trim() === '') {
    throw new Error('ProductId cannot be empty');
  }
  return raw as ProductId;
};
```

See [entity-template.md](./references/entity-template.md) for the full file template.

## Creating Value Objects

Value objects have **no identity** — two value objects are equal if all their properties are equal.

### Rules

- No `id` property
- All properties are `readonly`
- Factory function validates and returns `Result<T>`
- Equality is checked by comparing all field values
- Prefer primitive values or plain objects — avoid classes

See [value-object-template.md](./references/value-object-template.md) for the full file template.

## Creating Repository Ports

Repository ports define the contract between the domain and the infrastructure. Use an **interface + InjectionToken** pattern — never abstract classes.

### Rules

- Define a TypeScript `interface` with the repository contract
- Export an `InjectionToken` for Angular's dependency injection
- Standard CRUD methods return `Promise<Result<T>>`
- `InjectionToken` is the only Angular import allowed in the domain layer

See [repository-port-template.md](./references/repository-port-template.md) for the full file template.

## Key Principles

- **Pure TypeScript only** — no Angular imports in domain (exception: `InjectionToken` in port files)
- **All properties readonly** — entities and value objects are immutable
- **Factory functions validate and return `Result<T>`** — no throwing in domain logic
- **No class methods on entities** — use standalone pure functions exported from the same file
- **Branded IDs prevent type confusion** — each aggregate root has its own ID brand
- **Interface + InjectionToken for dependency inversion** — the domain defines the contract; infrastructure implements it

## Verification Checklist

Before committing domain artifacts, verify:

- [ ] No `import` statements reference Angular libraries (except `InjectionToken` in port files)
- [ ] No `import` statements reference infrastructure or HTTP modules
- [ ] All entity and value object properties are `readonly`
- [ ] All factory functions return `Result<T>` on both success and failure paths
- [ ] Entity logic lives in standalone functions, not class methods
- [ ] Each aggregate root has its own branded ID type
- [ ] Repository port uses `interface` + `InjectionToken`, not an abstract class
- [ ] Unit tests exist for factory validation logic

## Related Skills

- `angular-hexagonal-init` — scaffolds the full hexagonal project structure
- `angular-application-layer` — creates use cases and CQRS handlers that consume domain entities
- `angular-infrastructure-layer` — implements repository ports and maps API DTOs to domain entities
- `angular-state-management` — manages application state using domain entities
