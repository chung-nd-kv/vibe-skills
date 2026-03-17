# Angular Code Review Patterns

Apply these checks when reviewing Angular 17+ code.

## Component Patterns

- `ChangeDetectionStrategy.OnPush` — always required
- `inject()` function — never constructor injection
- `@if`/`@for`/`@switch` — never `*ngIf`/`*ngFor`/`[ngSwitch]`
- Signal inputs: `input()` / `input.required()` — never `@Input()`
- Signal outputs: `output()` — never `@Output()` / `EventEmitter`
- No `standalone: true` — default since Angular 17
- `host: { class: '...' }` for host class binding

## Hexagonal Layer Dependencies

| Layer          | Can Depend On       | Violation Example                      |
| -------------- | ------------------- | -------------------------------------- |
| Domain         | Nothing             | Importing HttpClient in domain         |
| Application    | Domain              | Importing a Component in application   |
| Infrastructure | Domain, Application | OK                                     |
| Presentation   | Application         | Importing directly from infrastructure |

## State & Reactivity Patterns

- Signals for UI state (loading, error, filters)
- RxJS for async event streams (Subject + operator + takeUntilDestroyed)
- `takeUntilDestroyed(this.destroyRef)` — always pass DestroyRef explicitly in ngOnInit
- `toSignal(obs.pipe(startWith(value)))` — always use startWith
- No `BehaviorSubject` for simple state (use signals)

## Domain Patterns

- Branded types for IDs: `type EntityId = number & { __brand: 'EntityId' }`
- All domain properties `readonly`
- Factory functions return `Result<T>`
- Static mapper classes (no `@Injectable()`)
- `Result<T>` for all fallible operations

## Import Ordering

1. `@angular/*`
2. Third-party (rxjs, etc.)
3. Project aliases (`@app/*`)
4. Relative imports

## File Size Guidelines

- Components: ~200 lines max
- Services: ~300 lines max
- Use Cases: ~100 lines max
- Domain files: ~200 lines max
- Consider splitting if exceeding these limits
