# Grid Component Template

Full annotated template for a pure presentational data grid component that receives data via signal inputs and emits user actions via outputs.

## Variable Substitution

| Placeholder | Format | Example |
|---|---|---|
| `{{EntityName}}` | PascalCase | `Product` |
| `{{entityName}}` | camelCase | `product` |
| `{{entity-name}}` | kebab-case | `product` |

## Event Type Interfaces

Define these interfaces inline in the component file when your component library does not provide them. Replace with your library's built-in event types when available.

```typescript
// Inline event type interfaces — replace with your library's types if available.
// For example:
//   Angular Material: PageEvent from @angular/material/paginator
//   PrimeNG: TablePageEvent, TableSortEvent from primeng/table
//   Syncfusion: PageEventArgs, SortEventArgs from @syncfusion/ej2-angular-grids

interface TablePageChangeEvent {
  page: number;
  pageSize: number;
}

interface TableRowClickEvent<T> {
  row: T;
}

interface TableSortChangeEvent {
  field: string;
  direction: 'asc' | 'desc';
}
```

## TypeScript — `{{entity-name}}-grid.component.ts`

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  input,
  output,
  computed,
} from '@angular/core';

// Import your UI library's table, pagination, badge, and empty state components:
// import { MatTableModule } from '@angular/material/table';
// import { MatPaginatorModule, PageEvent } from '@angular/material/paginator';
// import { MatButtonModule } from '@angular/material/button';
// Or your own shared UI components

// Import translation pipe if your project uses i18n:
// import { TranslatePipe } from '@your-org/localization';

import { {{EntityName}} } from '../../../domain/entities/{{entity-name}}.entity';

// ─── Event Type Interfaces ────────────────────────────────────────────────────
// Define inline when your library does not export these types.
// Replace with your library's event types when available.
interface TablePageChangeEvent {
  page: number;
  pageSize: number;
}

interface TableRowClickEvent<T> {
  row: T;
}

interface TableSortChangeEvent {
  field: string;
  direction: 'asc' | 'desc';
}

@Component({
  selector: 'app-{{entity-name}}-grid',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    // Add your UI library's components here:
    // MatTableModule, MatPaginatorModule, MatButtonModule, TranslatePipe, etc.
  ],
  templateUrl: './{{entity-name}}-grid.component.html',
  styleUrl: './{{entity-name}}-grid.component.scss',
})
export class {{EntityName}}GridComponent {
  // ─── Required Inputs ──────────────────────────────────────────────────────
  // data is always required — the grid cannot render without it
  readonly data = input.required<{{EntityName}}[]>();

  // ─── Optional Inputs ──────────────────────────────────────────────────────
  // Provide sensible defaults for all optional inputs

  // Whether the parent is fetching data
  readonly loading = input<boolean>(false);

  // Total item count for pagination (may differ from data.length when server-side paging)
  readonly totalItems = input<number>(0);

  // Current page number (1-based)
  readonly currentPage = input<number>(1);

  // Number of items per page
  readonly pageSize = input<number>(20);

  // True if ANY items exist for this entity type (regardless of current filter/search).
  // Used to differentiate "first-time setup" empty state from "no results" empty state.
  readonly hasAnyItem = input<boolean>(false);

  // True if a parent/related entity exists (e.g., a category has been created).
  // Used to differentiate "create the parent first" empty state from other states.
  readonly hasRelatedData = input<boolean>(true);

  // ─── Outputs ──────────────────────────────────────────────────────────────
  // Grid components emit events — they never handle business logic internally.
  // The parent (page component) reacts to these events.

  // User clicked a row to view/select it
  readonly rowClick = output<{{EntityName}}>();

  // User changed the page or page size
  readonly pageChange = output<TablePageChangeEvent>();

  // User clicked a column header to sort
  readonly sortChange = output<TableSortChangeEvent>();

  // User clicked the edit action button on a row
  readonly edit = output<{{EntityName}}>();

  // User clicked the delete action button on a row
  readonly delete = output<{{EntityName}}>();

  // User clicked the "Add" button in an empty state or header
  readonly add = output<void>();

  // ─── Computed Signals ─────────────────────────────────────────────────────

  // Total page count for pagination display
  readonly totalPages = computed(() =>
    this.pageSize() > 0
      ? Math.ceil(this.totalItems() / this.pageSize())
      : 0
  );

  // Human-readable pagination summary (e.g., "1–20 of 47")
  readonly paginationText = computed(() => {
    const total = this.totalItems();
    if (total === 0) return '0 items';
    const start = (this.currentPage() - 1) * this.pageSize() + 1;
    const end = Math.min(this.currentPage() * this.pageSize(), total);
    return `${start}–${end} of ${total}`;
  });

  // Whether the current view has no data at all
  readonly isEmpty = computed(() =>
    !this.loading() && this.data().length === 0
  );

  // ─── Event Handlers ───────────────────────────────────────────────────────
  // Handlers translate library-specific events into domain output events.

  onRowClick(event: TableRowClickEvent<{{EntityName}}>): void {
    this.rowClick.emit(event.row);
  }

  // If your library emits the row directly (not wrapped in an event object):
  onRowClickDirect(item: {{EntityName}}): void {
    this.rowClick.emit(item);
  }

  onPageChange(event: TablePageChangeEvent): void {
    this.pageChange.emit(event);
  }

  onSortChange(event: TableSortChangeEvent): void {
    this.sortChange.emit(event);
  }

  onEdit(item: {{EntityName}}, domEvent: Event): void {
    // IMPORTANT: stopPropagation prevents the row click from also firing
    // when the user clicks an action button inside a row.
    domEvent.stopPropagation();
    this.edit.emit(item);
  }

  onDelete(item: {{EntityName}}, domEvent: Event): void {
    domEvent.stopPropagation();
    this.delete.emit(item);
  }

  onAdd(): void {
    this.add.emit();
  }

  // ─── Track Function ───────────────────────────────────────────────────────
  // Always provide a trackBy function for @for loops.
  // This enables Angular to reuse DOM nodes when the data array changes,
  // dramatically improving performance for large or frequently updated lists.
  trackById(index: number, item: {{EntityName}}): string {
    return item.id;
  }
}
```

## HTML Template — `{{entity-name}}-grid.component.html`

```html
<!-- Loading state -->
@if (loading()) {
  <!-- Replace with your library's loading/spinner component -->
  <div class="loading-indicator" aria-label="Loading...">
    Loading...
  </div>
}

<!-- Data table — shown when data is available -->
@if (!loading() && data().length > 0) {
  <!-- Replace with your library's table component -->
  <!-- Example with native HTML table — substitute with mat-table, p-table, ejs-grid, etc. -->
  <table class="data-table" aria-label="{{EntityName}} list">
    <thead>
      <tr>
        <th (click)="onSortChange({ field: 'name', direction: 'asc' })">Name</th>
        <th>Description</th>
        <th>Status</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      @for (item of data(); track trackById($index, item)) {
        <tr (click)="onRowClickDirect(item)" class="data-row">
          <td>{{ item.name }}</td>
          <td>{{ item.description }}</td>
          <td>
            <!-- Replace with your library's badge/tag component -->
            <span class="badge" [class.active]="item.isActive">
              {{ item.isActive ? 'Active' : 'Inactive' }}
            </span>
          </td>
          <td class="actions-cell">
            <!-- stopPropagation is handled in the component method -->
            <button
              type="button"
              aria-label="Edit"
              (click)="onEdit(item, $event)"
            >
              Edit
            </button>
            <button
              type="button"
              aria-label="Delete"
              (click)="onDelete(item, $event)"
            >
              Delete
            </button>
          </td>
        </tr>
      }
    </tbody>
  </table>

  <!-- Pagination -->
  <!-- Replace with your library's pagination component -->
  <div class="pagination-bar">
    <span class="pagination-text">{{ paginationText() }}</span>
    <!-- Your library's paginator here -->
    <!-- (pageChange)="onPageChange($event)" -->
  </div>
}

<!-- Empty states — three distinct scenarios -->
@if (isEmpty()) {

  <!-- Scenario 1: No related/parent data exists yet -->
  @if (!hasRelatedData()) {
    <div class="empty-state">
      <h3>No parent data found</h3>
      <p>Create a parent record first before adding items here.</p>
    </div>

  <!-- Scenario 2: First-time setup — no items of this type exist anywhere -->
  } @else if (!hasAnyItem()) {
    <div class="empty-state">
      <h3>No items yet</h3>
      <p>Get started by creating your first item.</p>
      <button type="button" (click)="onAdd()">
        Add First Item
      </button>
    </div>

  <!-- Scenario 3: Items exist, but current search/filter returns nothing -->
  } @else {
    <div class="empty-state">
      <h3>No results found</h3>
      <p>Try adjusting your search or filter criteria.</p>
    </div>
  }

}
```

## Empty State Design Guide

The three empty state scenarios reflect different user contexts:

| Scenario | `hasRelatedData()` | `hasAnyItem()` | Message intent |
|---|---|---|---|
| No parent/related entity | `false` | any | Guide user to create prerequisite data |
| First-time setup | `true` | `false` | Onboarding — encourage creating the first item |
| No search results | `true` | `true` | Search/filter refinement — no destructive action shown |

Only show the "Add" button in the first-time setup scenario. Never show it when a search produced no results — the user needs to clear their filter, not create a new item.

## Checklist

- [ ] `standalone: true`
- [ ] `ChangeDetectionStrategy.OnPush`
- [ ] `data = input.required<{{EntityName}}[]>()` — required, not optional
- [ ] All other inputs have sensible defaults
- [ ] All outputs use `output<T>()` — no `@Output() EventEmitter`
- [ ] `computed totalPages` and `computed paginationText` derived from inputs
- [ ] `computed isEmpty` — `!loading() && data().length === 0`
- [ ] Three empty state scenarios handled: no related data, first-time setup, no search results
- [ ] `onEdit` and `onDelete` call `domEvent.stopPropagation()`
- [ ] `@for` loops use `track trackById($index, item)` — not `track item.id` directly
- [ ] No business logic in the component — only event emission
- [ ] Component exported from `presentation/components/index.ts`
