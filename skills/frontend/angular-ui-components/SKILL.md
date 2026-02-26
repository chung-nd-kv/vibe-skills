---
name: angular-ui-components
description: Creates Angular presentation layer UI components following hexagonal architecture patterns including page components, reactive form components for create and edit, and data grid components with signal inputs and OnPush change detection.
---

# Angular UI Components

Creates presentation layer components for Angular hexagonal architecture — pages, forms, and data grids.

## Architecture Position

```
UI Layer (Angular Components)  <-- YOU ARE HERE
      |
Application Layer (Use Cases / Commands / Queries)
      |
Domain Layer
      |
Infrastructure Layer (implements domain ports)
```

The UI layer:
- Is the outermost layer — the entry point for all user interactions
- Depends on state services and domain types only
- Does NOT inject repositories or use cases directly
- Communicates exclusively through state services

## Universal Component Rules

All components in the presentation layer follow these rules:

1. **`standalone: true`** — No NgModules; each component declares its own imports
2. **`ChangeDetectionStrategy.OnPush`** — Improves performance; only re-renders on signal/input changes
3. **Signal inputs** — Use `input()` and `input.required()` (not `@Input()` decorator)
4. **Signal outputs** — Use `output()` (not `@Output()` + `EventEmitter`)
5. **New control flow** — Use `@if`, `@for`, `@switch` (not `*ngIf`, `*ngFor`, `*ngSwitch`)
6. **`inject()` for DI** — Use `inject()` function, not constructor injection
7. **Separate template and styles** — Use `templateUrl` and `styleUrl`, not inline
8. **Typed Reactive Forms** — Always type `FormGroup<T>` and `FormControl<T>` explicitly

## Creating Page Components

Page components are the route entry points that orchestrate state services and child components.

### File Location

```
<module-path>/src/lib/presentation/pages/<module-name>.component.ts
<module-path>/src/lib/presentation/pages/<module-name>.component.html
<module-path>/src/lib/presentation/pages/<module-name>.component.scss
```

### Key Patterns

- **Inject state service** via `inject()` — never inject use cases or repositories directly
- **Debounced search** — use `searchSubject` + `debounceTime` + `takeUntilDestroyed`
- **Computed signals** — expose `data`, `loading`, `error`, `hasData`, `isEmpty` to template
- **`effect()` for side effects** — use sparingly, only for error logging or non-reactive side effects
- **`ngOnInit`** — call `state.load()` to trigger initial data fetch
- **Event handlers** — `onSearch`, `onPageChange`, `onRowClick`, `onCreate`, `onEdit`, `onDelete`
- **`trackById`** — always provide a track function for `@for` loops

### Code Skeleton

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  OnInit,
  inject,
  computed,
  effect,
} from '@angular/core';
import { Subject } from 'rxjs';
import { debounceTime, takeUntilDestroyed } from 'rxjs/operators';
// Import your UI library's table, button, badge, empty components
// Import your child components (grid, form)
import { {{ModuleName}}StateService } from '../services/{{module-name}}.state.service';
import { {{EntityName}} } from '../../domain/entities/{{entity-name}}.entity';

@Component({
  selector: 'app-{{page-name}}',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    // YourTableComponent, YourButtonComponent, etc.
    {{EntityName}}GridComponent,
  ],
  templateUrl: './{{page-name}}.component.html',
  styleUrl: './{{page-name}}.component.scss',
})
export class {{PageName}}Component implements OnInit {
  private readonly state = inject({{ModuleName}}StateService);

  // Expose state signals to template
  readonly data = computed(() => this.state.items());
  readonly loading = computed(() => this.state.loading());
  readonly error = computed(() => this.state.error());
  readonly hasData = computed(() => this.data().length > 0);
  readonly isEmpty = computed(() => !this.loading() && this.data().length === 0);

  // Search
  private readonly searchSubject = new Subject<string>();

  constructor() {
    this.searchSubject.pipe(
      debounceTime(300),
      takeUntilDestroyed(),
    ).subscribe(term => this.state.search(term));

    // Use effect sparingly — only for logging or non-reactive side effects
    effect(() => {
      const err = this.error();
      if (err) console.error('[{{PageName}}]', err);
    });
  }

  ngOnInit(): void {
    this.state.load();
  }

  onSearch(term: string): void {
    this.searchSubject.next(term);
  }

  onPageChange(page: number): void {
    this.state.setPage(page);
  }

  onRowClick(item: {{EntityName}}): void {
    // navigate or open detail
  }

  onCreate(): void {
    // open create modal / navigate to create page
  }

  onEdit(item: {{EntityName}}): void {
    this.state.selectItem(item);
    // open edit modal
  }

  onDelete(item: {{EntityName}}): void {
    this.state.delete(item.id);
  }

  trackById(index: number, item: {{EntityName}}): string {
    return item.id;
  }
}
```

See [page-template.md](./references/page-template.md) for the full annotated template.

## Creating Form Components

Form components are designed as modal content, supporting both create and edit modes via a single component.

### File Location

```
<module-path>/src/lib/presentation/components/<entity-name>-form/<entity-name>-form.component.ts
<module-path>/src/lib/presentation/components/<entity-name>-form/<entity-name>-form.component.html
<module-path>/src/lib/presentation/components/<entity-name>-form/<entity-name>-form.component.scss
<module-path>/src/lib/presentation/components/<entity-name>-form/index.ts
```

### Key Patterns

- **`mode = input.required<'create' | 'edit'>()`** — single component handles both modes
- **Optional entity input** — `entity = input<{{EntityName}} | null>(null)` for edit pre-population
- **Optional entityId input** — `entityId = input<string | null>(null)` when only ID is needed
- **Typed `FormGroup`** — always type the group and each control explicitly
- **`toSignal(form.statusChanges)`** — convert form status observable to signal for computed
- **`computed canSubmit`** — derive from form validity + submitting state
- **`viewChild` for focus** — auto-focus the first field on open
- **`onSubmit`** — dispatches to `handleCreate` or `handleUpdate` based on mode
- **Modal control** — close modal with a return value after successful submission

### Code Skeleton

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  OnInit,
  inject,
  input,
  output,
  computed,
  viewChild,
  ElementRef,
} from '@angular/core';
import { ReactiveFormsModule, FormGroup, FormControl, Validators } from '@angular/forms';
import { toSignal } from '@angular/core/rxjs-interop';
// Import your UI library's button and form control components
// Import your modal control service/token if applicable
import { {{ModuleName}}StateService } from '../../services/{{module-name}}.state.service';
import { {{EntityName}} } from '../../../domain/entities/{{entity-name}}.entity';

interface {{EntityName}}Form {
  name: FormControl<string>;
  // add more typed controls here
}

@Component({
  selector: 'app-{{entity-name}}-form',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    ReactiveFormsModule,
    // YourButtonComponent, YourFormControlComponent, etc.
  ],
  templateUrl: './{{entity-name}}-form.component.html',
  styleUrl: './{{entity-name}}-form.component.scss',
})
export class {{EntityName}}FormComponent implements OnInit {
  // Inputs
  readonly mode = input.required<'create' | 'edit'>();
  readonly entity = input<{{EntityName}} | null>(null);
  readonly entityId = input<string | null>(null);

  // Outputs
  readonly saved = output<{{EntityName}}>();
  readonly cancelled = output<void>();

  private readonly state = inject({{ModuleName}}StateService);
  // private readonly modalControl = inject(ModalControl); // inject your modal control if applicable

  // Focus management
  readonly firstField = viewChild<ElementRef>('firstField');

  // Form
  readonly form = new FormGroup<{{EntityName}}Form>({
    name: new FormControl('', { nonNullable: true, validators: [Validators.required, Validators.maxLength(100)] }),
  });

  // Reactive form status as signal
  private readonly formStatus = toSignal(this.form.statusChanges);

  // Submitting state
  private submitting = false;

  // Computed
  readonly canSubmit = computed(() =>
    this.formStatus() === 'VALID' && !this.submitting
  );

  readonly isEditMode = computed(() => this.mode() === 'edit');

  ngOnInit(): void {
    // Pre-populate form in edit mode
    const existing = this.entity();
    if (existing && this.mode() === 'edit') {
      this.form.patchValue({ name: existing.name });
    }

    // Auto-focus first field
    setTimeout(() => this.firstField()?.nativeElement.focus(), 50);
  }

  onSubmit(): void {
    if (!this.canSubmit()) return;
    this.form.markAllAsTouched();
    if (this.form.invalid) return;

    if (this.mode() === 'create') {
      this.handleCreate();
    } else {
      this.handleUpdate();
    }
  }

  private async handleCreate(): Promise<void> {
    this.submitting = true;
    try {
      const result = await this.state.create(this.form.getRawValue());
      if (result) {
        this.saved.emit(result);
        // this.modalControl.close(result); // close modal with return value
      }
    } finally {
      this.submitting = false;
    }
  }

  private async handleUpdate(): Promise<void> {
    const id = this.entityId() ?? this.entity()?.id;
    if (!id) return;
    this.submitting = true;
    try {
      const result = await this.state.update(id, this.form.getRawValue());
      if (result) {
        this.saved.emit(result);
        // this.modalControl.close(result); // close modal with return value
      }
    } finally {
      this.submitting = false;
    }
  }

  onCancel(): void {
    this.cancelled.emit();
    // this.modalControl.close(null); // close without result
  }
}
```

See [form-template.md](./references/form-template.md) for the full annotated template.

## Creating Grid Components

Grid components are pure presentational components — they receive data via inputs and emit user actions via outputs.

### File Location

```
<module-path>/src/lib/presentation/components/<entity-name>-grid/<entity-name>-grid.component.ts
<module-path>/src/lib/presentation/components/<entity-name>-grid/<entity-name>-grid.component.html
<module-path>/src/lib/presentation/components/<entity-name>-grid/<entity-name>-grid.component.scss
<module-path>/src/lib/presentation/components/<entity-name>-grid/index.ts
```

### Key Patterns

- **`data = input.required<Entity[]>()`** — always required
- **Optional inputs** — `loading`, `totalItems`, `currentPage`, `pageSize`, `hasAnyItem`, `hasRelatedData`
- **Outputs** — `rowClick`, `pageChange`, `sortChange`, `edit`, `delete`, `add`
- **Computed** — `totalPages`, `paginationText`
- **Three empty state scenarios**:
  1. First-time setup — no items ever existed (`!hasAnyItem()`)
  2. No items in category — items exist elsewhere but filter returns none
  3. No search results — active search query returns nothing
- **`trackById`** — always provide a track function for `@for` loops

### Code Skeleton

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  inject,
  input,
  output,
  computed,
} from '@angular/core';
// Import your UI library's table, button, badge, pagination, empty state components

import { {{EntityName}} } from '../../../domain/entities/{{entity-name}}.entity';

// Event type interfaces (replace with your library's types if available)
interface TablePageChangeEvent { page: number; pageSize: number; }
interface TableRowClickEvent<T> { row: T; }
interface TableSortChangeEvent { field: string; direction: 'asc' | 'desc'; }

@Component({
  selector: 'app-{{entity-name}}-grid',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    // YourTableComponent, YourPaginationComponent, YourEmptyStateComponent, etc.
  ],
  templateUrl: './{{entity-name}}-grid.component.html',
  styleUrl: './{{entity-name}}-grid.component.scss',
})
export class {{EntityName}}GridComponent {
  // Required inputs
  readonly data = input.required<{{EntityName}}[]>();

  // Optional inputs
  readonly loading = input<boolean>(false);
  readonly totalItems = input<number>(0);
  readonly currentPage = input<number>(1);
  readonly pageSize = input<number>(20);
  readonly hasAnyItem = input<boolean>(false); // true if any items exist (regardless of current filter)
  readonly hasRelatedData = input<boolean>(false); // true if parent/related entity exists

  // Outputs
  readonly rowClick = output<{{EntityName}}>();
  readonly pageChange = output<TablePageChangeEvent>();
  readonly sortChange = output<TableSortChangeEvent>();
  readonly edit = output<{{EntityName}}>();
  readonly delete = output<{{EntityName}}>();
  readonly add = output<void>();

  // Computed
  readonly totalPages = computed(() =>
    Math.ceil(this.totalItems() / this.pageSize())
  );

  readonly paginationText = computed(() => {
    const start = (this.currentPage() - 1) * this.pageSize() + 1;
    const end = Math.min(this.currentPage() * this.pageSize(), this.totalItems());
    return `${start}–${end} of ${this.totalItems()}`;
  });

  // Event handlers
  onRowClick(event: TableRowClickEvent<{{EntityName}}>): void {
    this.rowClick.emit(event.row);
  }

  onPageChange(event: TablePageChangeEvent): void {
    this.pageChange.emit(event);
  }

  onSortChange(event: TableSortChangeEvent): void {
    this.sortChange.emit(event);
  }

  onEdit(item: {{EntityName}}, domEvent: Event): void {
    domEvent.stopPropagation(); // prevent row click from firing
    this.edit.emit(item);
  }

  onDelete(item: {{EntityName}}, domEvent: Event): void {
    domEvent.stopPropagation();
    this.delete.emit(item);
  }

  onAdd(): void {
    this.add.emit();
  }

  trackById(index: number, item: {{EntityName}}): string {
    return item.id;
  }
}
```

See [grid-template.md](./references/grid-template.md) for the full annotated template.

## Component Library Integration Note

These patterns work with **any** Angular component library. The architecture is library-agnostic.

Substitute your library's components where indicated in the templates:

| Pattern need | Angular Material | PrimeNG | Syncfusion | Custom |
|---|---|---|---|---|
| Data table | `mat-table` | `p-table` | `ejs-grid` | `<your-table>` |
| Button | `mat-button` | `p-button` | `ejs-button` | `<your-button>` |
| Badge/Tag | `mat-chip` | `p-badge` | `ejs-badge` | `<your-badge>` |
| Form control | `mat-form-field` | `p-inputtext` | `ejs-textbox` | `<your-input>` |
| Empty state | Custom | `p-empty` | Custom | `<your-empty>` |

The patterns — signals, OnPush, standalone, typed forms, computed — are library-agnostic and remain constant regardless of which component library you use.

## Key Principles

1. **Inject state, not use cases** — Page components orchestrate via state services only
2. **Pure presentational grids** — Grid components receive data and emit events; no business logic
3. **Single form, dual mode** — One form component handles both create and edit via `mode` input
4. **Signals everywhere** — `input()`, `output()`, `computed()`, `toSignal()` for reactivity
5. **OnPush always** — Every component uses `ChangeDetectionStrategy.OnPush`
6. **Standalone always** — No NgModules; each component manages its own imports
7. **New control flow** — `@if`, `@for`, `@switch` only; never structural directives
8. **`trackById` always** — Every `@for` loop must have a track function
9. **`stopPropagation` in nested actions** — Action buttons in rows must stop event bubbling
10. **Three empty states** — Design for: no items ever, no items in category, no search results

## Verification Checklist

Before committing a presentation layer component, verify:

- [ ] `standalone: true` — no NgModule declarations
- [ ] `ChangeDetectionStrategy.OnPush`
- [ ] Signal inputs: `input()` or `input.required()` — no `@Input()` decorator
- [ ] Signal outputs: `output()` — no `@Output() EventEmitter`
- [ ] `inject()` for dependency injection — no constructor parameters
- [ ] `@if`, `@for`, `@switch` — no structural directives (`*ngIf`, `*ngFor`)
- [ ] Separate `templateUrl` and `styleUrl` — no inline template/styles
- [ ] Page components inject only state services — no repositories or use cases directly
- [ ] Form components support both `'create'` and `'edit'` modes
- [ ] Grid components are purely presentational — no business logic, only emit events
- [ ] Every `@for` loop has a `track` expression
- [ ] Action buttons in grid rows call `$event.stopPropagation()`

## Related Skills

- `angular-hexagonal-init` — scaffolds the full hexagonal project structure
- `angular-domain-modeling` — creates domain entities consumed by UI components
- `angular-application-layer` — creates use cases dispatched by state services
- `angular-infrastructure-layer` — implements repository ports and data access
- `angular-state-management` — creates the state services that page components inject
