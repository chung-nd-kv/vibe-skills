---
name: angular-component
description: Angular 17+ component patterns with signal-based I/O, OnPush change detection, native control flow (@if/@for/@switch), and inject() function. Use when creating, modifying, or reviewing Angular components. ALWAYS use when writing .component.ts files, editing @Component decorators, or configuring component imports.
---

# Angular Component Patterns

## Component Declaration

Components are standalone by default in Angular 17+. Do NOT set `standalone: true`.

```typescript
@Component({
  selector: 'app-entity-list',
  imports: [CommonModule, RouterModule],
  templateUrl: './entity-list.component.html',
  styleUrl: './entity-list.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class EntityListComponent {
  // 1. Injected dependencies
  private readonly service = inject(EntityService);
  private readonly router = inject(Router);

  // 2. Signal inputs
  readonly entityId = input.required<string>();
  readonly status = input<string>('active');

  // 3. Signal outputs
  readonly selected = output<Entity>();

  // 4. Signals for reactive state
  readonly items = signal<Entity[]>([]);
  readonly loading = signal(false);

  // 5. Computed values
  readonly filteredItems = computed(() =>
    this.items().filter(i => this.status() === 'all' || i.status === this.status())
  );
}
```

## Signal Inputs (not @Input)

```typescript
// Required input
readonly entityId = input.required<number>();

// Optional with default
readonly pageSize = input(20);

// Transform
readonly id = input(0, { transform: numberAttribute });

// Alias
readonly label = input('', { alias: 'buttonLabel' });
```

## Signal Outputs (not @Output)

```typescript
readonly selected = output<Entity>();
readonly closed = output<void>();

// Emit
this.selected.emit(entity);
this.closed.emit();
```

## View Queries (not @ViewChild)

```typescript
readonly nameInput = viewChild<ElementRef<HTMLInputElement>>('nameInput');
readonly items = viewChildren(ItemComponent);

// Usage
this.nameInput()?.nativeElement.focus();
```

## Control Flow (not *ngIf/*ngFor)

```html
@if (loading()) {
  <app-spinner />
} @else if (error()) {
  <app-error [message]="error()" />
} @else {
  @for (item of items(); track item.id) {
    <app-item [data]="item" (click)="onSelect(item)" />
  } @empty {
    <p>No items found</p>
  }
}

@switch (status()) {
  @case ('active') { <span class="badge-active">Active</span> }
  @case ('inactive') { <span class="badge-inactive">Inactive</span> }
  @default { <span>Unknown</span> }
}
```

## inject() Function (not constructor injection)

```typescript
// Always use inject()
private readonly http = inject(HttpClient);
private readonly destroyRef = inject(DestroyRef);
private readonly route = inject(ActivatedRoute);
private readonly modalControl = inject(ModalControl, { optional: true });
```

## Host Metadata

```typescript
@Component({
  host: {
    class: 'page-content',
    '[class.is-loading]': 'loading()',
    '(window:resize)': 'onResize($event)',
  },
})
```

## Import Order

1. `@angular/*`
2. Third-party libraries (rxjs, etc.)
3. Project aliases (`@app/*`)
4. Relative imports

## Key Rules

- Always `ChangeDetectionStrategy.OnPush`
- Always `inject()`, never constructor injection
- Signal inputs: `input()` / `input.required()`, not `@Input()`
- Signal outputs: `output()`, not `@Output()` / `EventEmitter`
- Native control flow: `@if`/`@for`/`@switch`, not `*ngIf`/`*ngFor`
- Separate template and style files (except very simple components)

For page component patterns, see [references/component-patterns.md](references/component-patterns.md).
