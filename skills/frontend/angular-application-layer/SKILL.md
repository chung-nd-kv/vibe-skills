---
name: angular-application-layer
description: Creates application layer artifacts for Angular hexagonal architecture including DTOs, mapper classes, command use cases for write operations, and query use cases for read operations following CQRS patterns.
---

# Angular Application Layer

Creates application layer artifacts that orchestrate domain objects. Depends only on the Domain layer.

## Architecture Position

The Application layer sits between the Domain and the outer layers (Infrastructure, Presentation):

```
Presentation (Components, Pages, State)
       │
       ▼
   Application (UseCases, Queries, DTOs, Mappers)
       │
       ▼
     Domain (Entities, Value Objects, Repository Ports)
       │
       ▲
Infrastructure (Repository Implementations, API Clients)
```

**The dependency rule**: The Application layer depends only on Domain. It never imports from Infrastructure or Presentation. Infrastructure implements Domain interfaces (Ports & Adapters pattern).

---

## Creating DTOs

DTOs are plain TypeScript interfaces that describe the data shape exchanged with external systems (typically a backend API).

**PascalCase convention**: Properties use PascalCase by default to match typical API contracts. This is configurable — if your API uses camelCase or snake_case, adjust accordingly and update the mapper.

### Three DTO Types

| DTO | Purpose |
|-----|---------|
| `{{EntityName}}Dto` | Response shape from API (read) |
| `Create{{EntityName}}Dto` | Payload sent to API for creation |
| `Update{{EntityName}}Dto` | Payload sent to API for update |

### API Response Wrappers

Most APIs wrap responses in an envelope object. Define these wrappers to match your API contract:

- `{{EntityName}}ListResponseDto` — wraps array responses
- `{{EntityName}}ResponseDto` — wraps single-entity responses
- `ApiMessageResponseDto` — wraps mutation responses (create/update/delete)

**File location**: `<module-path>/src/lib/application/dtos/<entity-name>.dto.ts`

See the full template in `references/dto-template.md`.

---

## Creating Mappers

Mappers translate between DTOs (API shapes) and domain entities (business objects). They are pure transformation classes with no side effects.

### Design Decisions

- **Static methods** — mappers are stateless, no need for instances
- **`toDomain`** — converts API DTO to domain entity: wraps primitives into branded types, parses ISO date strings to `Date`, validates via domain factory function
- **`toDto`** — converts domain entity to API DTO: strips branded types back to primitives, serializes `Date` to ISO strings
- **`toDomainList` / `toDtoList`** — convenience methods for collection mapping

### When toDomain Throws

The `toDomain` method throws an `Error` if the DTO fails domain validation (e.g., required field missing, invariant violated). This is intentional — a DTO that cannot be mapped to a valid domain entity indicates a contract mismatch between frontend and backend.

**File location**: `<module-path>/src/lib/application/mappers/<entity-name>.mapper.ts`

See the full template in `references/mapper-template.md`.

---

## CQRS Separation

The Application layer enforces strict separation between reads and writes:

| Side | Folder | Suffix | Purpose |
|------|--------|--------|---------|
| Write | `application/usecases/<entity>/` | `.usecase.ts` | Create, Update, Delete — modify state |
| Read | `application/queries/<entity>/` | `.query.ts` | Get, List — read-only, no side effects |

**Why separate?** Commands and queries have different concerns:
- Commands need validation, domain entity creation, and transactional writes
- Queries need efficiency and can bypass domain objects when reading directly from the repository
- Keeping them separate makes each class focused and easier to test

---

## Creating Command Use Cases

Command use cases handle write operations. Each use case is a single-responsibility class that executes one business command.

### Structure

Each command use case:
- Is decorated with `@Injectable()` for Angular DI
- Uses `inject()` (function-based DI, never constructor injection)
- Injects the repository via its `InjectionToken`
- Returns `Promise<Result<T>>` — never throws, always returns a Result

### Optional Base Class Pattern

You can define an abstract base class in your core library to enforce the contract:

```typescript
// Define in your core library:
export abstract class UseCaseBase<TRequest, TResponse> {
  abstract execute(request: TRequest): Promise<Result<TResponse>>;
}
```

Use cases then extend this base:

```typescript
export class CreateProductUseCase extends UseCaseBase<CreateProductDto, Product> {
  // ...
}
```

This is optional — a plain class with an `execute()` method works equally well if you prefer not to use inheritance.

### Execution Context Pattern

Many use cases need the current user or tenant context (e.g., to set `createdBy`, filter by tenant). Define an injection token in your core library:

```typescript
// Define in your core library:
export const APP_CONTEXT = new InjectionToken<AppContext>('AppContext');

export interface AppContext {
  userId: number;
  tenantId: number;
  // Add whatever your application needs
}
```

Then inject it in use cases that need it:

```typescript
private readonly context = inject(APP_CONTEXT);
```

The three standard command use cases are **Create**, **Update**, and **Delete**.

**File location**: `<module-path>/src/lib/application/usecases/<entity-name>/`

See the full template in `references/usecase-command-template.md`.

---

## Creating Query Use Cases

Query use cases handle read operations. They are read-only and must not produce side effects.

### Structure

Each query use case:
- Is decorated with `@Injectable()`
- Uses `inject()` for DI
- Injects the repository via its `InjectionToken`
- Returns `Promise<Result<T>>`
- Never modifies state

### Optional Base Class Pattern

Similar to commands, you can define a query base in your core library:

```typescript
// Define in your core library:
export abstract class QueryBase<TRequest, TResponse> {
  abstract execute(request: TRequest): Promise<Result<TResponse>>;
}
```

Or use a plain class without inheritance — both approaches are valid.

The two standard query use cases are **GetEntities** (list) and **GetEntity** (single by ID).

**File location**: `<module-path>/src/lib/application/queries/<entity-name>/`

See the full template in `references/usecase-query-template.md`.

---

## Barrel Export Hierarchy

Maintain index files at every level to enable clean imports:

```
application/
├── dtos/
│   ├── <entity-name>.dto.ts
│   └── index.ts                          # export * from './<entity-name>.dto'
├── mappers/
│   ├── <entity-name>.mapper.ts
│   └── index.ts                          # export * from './<entity-name>.mapper'
├── queries/
│   └── <entity-name>/
│       ├── get-<entity-name>s.query.ts
│       ├── get-<entity-name>.query.ts
│       └── index.ts                      # export * from each query file
├── usecases/
│   └── <entity-name>/
│       ├── create-<entity-name>.usecase.ts
│       ├── update-<entity-name>.usecase.ts
│       ├── delete-<entity-name>.usecase.ts
│       └── index.ts                      # export * from each usecase file
└── index.ts                              # export * from './dtos', './mappers', './queries', './usecases'
```

---

## Key Principles

1. **Application depends only on Domain** — never import from Infrastructure or Presentation
2. **DTOs are plain interfaces** — no class instances, no decorators, no runtime overhead
3. **Mappers are stateless** — static methods only, no side effects
4. **Commands modify state, queries read state** — enforce this separation strictly
5. **Always return `Result<T>`** — use cases never throw; failures are captured in Result
6. **Use `inject()` for DI** — function-based injection, not constructor injection
7. **File naming** — commands use `.usecase.ts` suffix, queries use `.query.ts` suffix

---

## Verification Checklist

- [ ] DTO interfaces defined for response, create, and update shapes
- [ ] API response wrappers defined to match backend contract
- [ ] Mapper has static `toDomain`, `toDto`, `toDomainList`, `toDtoList` methods
- [ ] `toDomain` uses domain factory function for validation
- [ ] Command use cases in `application/usecases/<entity>/` with `.usecase.ts` suffix
- [ ] Query use cases in `application/queries/<entity>/` with `.query.ts` suffix
- [ ] All use cases decorated with `@Injectable()`
- [ ] All use cases use `inject()` for DI (no constructor injection)
- [ ] All use cases return `Promise<Result<T>>`
- [ ] Barrel `index.ts` files created at each level
- [ ] Application `index.ts` exports everything
- [ ] No imports from Infrastructure or Presentation layers

---

## Related Skills

- `angular-hexagonal-init` — Sets up the module scaffold and folder structure
- `angular-domain-modeling` — Creates entities, value objects, and repository ports (Domain layer)
- `angular-infrastructure-layer` — Creates repository implementations and API clients (Infrastructure layer)
- `angular-state-management` — Creates state services in the Presentation layer
