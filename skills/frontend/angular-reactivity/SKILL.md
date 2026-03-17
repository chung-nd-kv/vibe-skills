---
name: angular-reactivity
description: Angular reactive patterns combining signals (signal(), computed(), effect(), linkedSignal()) for UI state with RxJS (Subject, switchMap, exhaustMap, race, combineLatest) for async event streams, plus interop (toSignal(), toObservable()). Use when working with reactive state, derived values, async event handling, or state management. ALWAYS use when creating state services or wiring Subject streams.
---

# Angular Reactivity — Signals + RxJS

Signals handle state; RxJS handles streams. They interop via `toSignal()` and `toObservable()`.

## When to Use What

| Use Signals                        | Use RxJS                                     |
| ---------------------------------- | -------------------------------------------- |
| UI state (loading, error, filters) | Async event streams (click, submit)          |
| Derived/computed values            | debounceTime, distinctUntilChanged           |
| Simple read/write state            | switchMap, exhaustMap, combineLatest         |
| Template bindings                  | Dialog handling (race, firstValueFrom)        |
| Component inputs/outputs           | Complex async orchestration                  |

## Signal Essentials

```typescript
// Writable
private readonly _items = signal<Entity[]>([]);
readonly items = this._items.asReadonly();

// Modify
this._items.set(newItems);
this._items.update(items => items.filter(i => i.id !== id));

// Computed (auto-updates)
readonly hasItems = computed(() => this._items().length > 0);

// linkedSignal (resets when source changes)
readonly selectedId = linkedSignal({
  source: this.items,
  computation: (items, prev) =>
    prev && items.some(i => i.id === prev.value) ? prev.value : items[0]?.id ?? null,
});
```

## Interop (Critical Patterns)

### toSignal() — Observable to Signal

```typescript
import { toSignal } from '@angular/core/rxjs-interop';
import { startWith } from 'rxjs';

// ALWAYS use startWith() to emit initial value
private readonly formStatus = toSignal(
  this.form.statusChanges.pipe(startWith(this.form.status)),
  { initialValue: this.form.status }
);
```

### toObservable() — Signal to Observable

```typescript
import { toObservable } from '@angular/core/rxjs-interop';

combineLatest([toObservable(this._keyword), toObservable(this._page)])
  .pipe(
    debounceTime(100),
    switchMap(() => this.fetchData()),
    takeUntilDestroyed(this.destroyRef)
  )
  .subscribe();
```

## RxJS Core Pattern: Subject + Operator + takeUntilDestroyed

```typescript
private readonly destroyRef = inject(DestroyRef);
private readonly deleteClick$ = new Subject<Entity>();

ngOnInit(): void {
  this.deleteClick$.pipe(
    switchMap(entity => from(this.handleDelete(entity))),
    takeUntilDestroyed(this.destroyRef)  // MUST pass destroyRef in ngOnInit
  ).subscribe();
}

onDelete(entity: Entity): void { this.deleteClick$.next(entity); }
```

**Never** use `takeUntilDestroyed()` without args in `ngOnInit()` — it will throw.

## Operator Selection

| Operator        | When                       | Example                           |
| --------------- | -------------------------- | --------------------------------- |
| `exhaustMap`    | Prevent concurrent ops     | Form submit, create, delete       |
| `switchMap`     | Cancel-and-restart         | Search, navigation, dialog open   |
| `debounceTime`  | Batch rapid changes        | Filter/search auto-reload         |
| `combineLatest` | Multiple signal sources    | State service reactive stream     |
| `race`          | First of competing streams | Dialog submit vs close            |
| `from()`        | Promise → Observable       | Wrap async handler for RxJS chain |

## Detailed Patterns

- Signal state service pattern: [references/signal-patterns.md](references/signal-patterns.md)
- RxJS event patterns (dialog, CRUD, deactivation): [references/rxjs-patterns.md](references/rxjs-patterns.md)
