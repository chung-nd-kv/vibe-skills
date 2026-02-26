---
name: angular-state-management
description: Creates Angular Signals-based state services for the presentation layer. Use when creating reactive UI state management, orchestrating use cases with signals, or bridging application layer use cases with Angular components.
---

# Angular State Management with Signals

Creates state services in the presentation layer using Angular Signals for reactive UI state management. Bridges use cases (application layer) with components.

## Architecture Position

The State service lives in the Presentation layer and depends on the Application layer:

```
Presentation (Components, Pages, State Services)
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

**Dependencies**: State services import from the Application layer (use cases, queries) and Domain layer (entity types). They never import from Infrastructure.

---

## Signals Pattern Overview

### Private Writable + Public Readonly Pattern

All mutable state is held in private writable signals. Public API exposes read-only views:

```typescript
private readonly _items = signal<readonly Entity[]>([]);
readonly items = this._items.asReadonly();
```

This enforces that only the state service itself can mutate state — components read but never write directly.

### Computed Signals for Derived State

Use `computed()` for any value that derives from other signals. Never duplicate state:

```typescript
readonly selectedItem = computed(() => {
  const id = this._selectedId();
  return this._items().find(i => i.id === id) ?? null;
});
```

### Action Method Pattern

Action methods are async, set loading/error signals, call use cases, and update state:

```typescript
async load(): Promise<void> {
  this._loading.set(true);
  this._error.set(null);
  try {
    const result = await this.getUseCase.execute();
    if (result.isSuccess && result.value) {
      this._items.set(result.value);
    } else {
      this._error.set(result.errorMessage ?? 'Failed to load');
    }
  } finally {
    this._loading.set(false);
  }
}
```

---

## Creating a State Service

### File Location

```
<module-path>/src/lib/presentation/state/<entity-name>.state.service.ts
```

### Private Writable Signals

Declare these as private class fields:

| Signal | Type | Purpose |
|--------|------|---------|
| `_items` | `signal<readonly Entity[]>([])` | Full list of entities |
| `_selectedId` | `signal<EntityId \| null>(null)` | Currently selected entity ID |
| `_loading` | `signal<boolean>(false)` | Loading indicator |
| `_error` | `signal<string \| null>(null)` | Current error message |
| `_hasAnyItem` | `signal<boolean>(false)` | Whether any items exist (e.g. from a lightweight check) |

### Public Readonly Signals

Expose read-only versions via `.asReadonly()`:

| Signal | Derived from |
|--------|-------------|
| `items` | `_items.asReadonly()` |
| `selectedId` | `_selectedId.asReadonly()` |
| `loading` | `_loading.asReadonly()` |
| `error` | `_error.asReadonly()` |
| `hasAnyItem` | `_hasAnyItem.asReadonly()` |

### Computed Signals

Derive these from existing signals using `computed()`:

| Computed | Logic |
|----------|-------|
| `selectedItem` | Find entity in `_items` by `_selectedId` |
| `activeItems` | Filter `_items` for active/enabled entities |
| `hasItems` | `_items().length > 0` |
| `total` | `_items().length` |

### Action Methods

Implement these async methods:

| Method | Returns | Purpose |
|--------|---------|---------|
| `load()` | `Promise<void>` | Initial load of all entities |
| `refresh()` | `Promise<void>` | Reload entities (e.g., after mutation) |
| `create(dto)` | `Promise<boolean>` | Create entity, return success flag |
| `update(dto)` | `Promise<boolean>` | Update entity, return success flag |
| `delete(id)` | `Promise<boolean>` | Delete entity, return success flag |
| `select(id)` | `void` | Set the selected entity ID |
| `getById(id)` | `Promise<Entity \| null>` | Fetch single entity by ID |
| `clearError()` | `void` | Reset `_error` to null |

See the full template in `references/state-service-template.md`.

---

## Modal Integration Pattern

State services can optionally manage opening create/edit modals. This keeps modal orchestration out of components and co-locates it with the state it affects.

Use `race()` and `firstValueFrom` to handle both a successful submit and a dismiss:

```typescript
// Generic modal pattern (adapt to your UI library's modal API)
async openCreateDialog(): Promise<Entity | null> {
  const modalRef = this.modalService.open(FormComponent, { inputs: { mode: 'create' } });
  // Use race() to handle both submit and close
  const result = await firstValueFrom(
    race(
      modalRef.afterSubmitted(),
      modalRef.afterClosed().pipe(map(() => null))
    )
  );
  if (result) await this.refresh();
  return result ?? null;
}
```

**Note**: Adapt `modalService.open`, `afterSubmitted()`, and `afterClosed()` to your UI library (Angular Material, PrimeNG, custom overlay, etc.).

---

## Provider Registration

Register state services at the route or feature level — never with `providedIn: 'root'`. This ensures each feature module gets its own isolated state instance:

```typescript
export const presentationProviders: Provider[] = [
  EntityStateService,
];
```

Wire this into your feature routes:

```typescript
{
  path: 'entities',
  providers: [...presentationProviders],
  component: EntityShellComponent,
}
```

---

## Key Principles

1. **Private writable, public readonly** — use `signal()` privately, expose via `.asReadonly()` only
2. **`computed()` for derived state** — never duplicate values with a second signal
3. **Actions return `Promise<boolean>`** — CRUD methods return a success flag; `load`/`refresh` return `void`
4. **No `providedIn: 'root'`** — always register at route/feature scope
5. **Inject use cases, not repositories** — state services depend on Application layer, not Infrastructure
6. **Use `inject()` for DI** — function-based injection, not constructor injection
7. **Single responsibility per signal** — each signal represents exactly one piece of state

---

## Verification Checklist

- [ ] State service file is at `presentation/state/<entity-name>.state.service.ts`
- [ ] All mutable signals are private (`_items`, `_loading`, `_error`, `_selectedId`, `_hasAnyItem`)
- [ ] All public signals are `.asReadonly()` wrappers
- [ ] Derived values use `computed()`, not duplicate signals
- [ ] CRUD action methods return `Promise<boolean>`
- [ ] `load()` and `refresh()` return `Promise<void>`
- [ ] `select()` and `clearError()` are synchronous void methods
- [ ] State service is NOT decorated with `providedIn: 'root'`
- [ ] State service is registered in `presentationProviders` array
- [ ] Only Application-layer use cases are injected (no repository injection)
- [ ] `inject()` used for all dependencies (no constructor injection)

---

## Related Skills

- `angular-hexagonal-init` — Sets up the module scaffold and folder structure
- `angular-domain-modeling` — Creates entities, value objects, and repository ports (Domain layer)
- `angular-application-layer` — Creates use cases and queries that the state service orchestrates
- `angular-infrastructure-layer` — Creates repository implementations (Infrastructure layer)
- `angular-ui-components` — Creates components that consume this state service
