# Page Component Template

Full annotated template for a presentation layer page component following Angular hexagonal architecture patterns.

## Variable Substitution

| Placeholder | Format | Example |
|---|---|---|
| `{{EntityName}}` | PascalCase | `Product` |
| `{{entityName}}` | camelCase | `product` |
| `{{PageName}}` | PascalCase | `ProductList` |
| `{{pageName}}` | camelCase | `productList` |
| `{{page-name}}` | kebab-case | `product-list` |
| `{{ModuleName}}` | PascalCase | `Inventory` |
| `{{moduleName}}` | camelCase | `inventory` |
| `{{module-name}}` | kebab-case | `inventory` |

## TypeScript — `{{page-name}}.component.ts`

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  OnInit,
  inject,
  computed,
  effect,
  signal,
} from '@angular/core';
import { Subject } from 'rxjs';
import { debounceTime, takeUntilDestroyed } from 'rxjs/operators';

// Import your UI library's components as needed:
// import { MatButtonModule } from '@angular/material/button';
// import { PrimeNGModule } from 'primeng/...';
// Or your own shared UI components

// Import translation pipe/service if your project uses i18n:
// import { TranslatePipe } from '@your-org/localization';

import { {{ModuleName}}StateService } from '../services/{{module-name}}.state.service';
import { {{EntityName}} } from '../../domain/entities/{{entity-name}}.entity';
import { {{EntityName}}GridComponent } from '../components/{{entity-name}}-grid/{{entity-name}}-grid.component';
import { {{EntityName}}FormComponent } from '../components/{{entity-name}}-form/{{entity-name}}-form.component';

@Component({
  selector: 'app-{{page-name}}',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    // List ALL imports your template uses — no NgModule to provide them
    {{EntityName}}GridComponent,
    {{EntityName}}FormComponent,
    // MatButtonModule, TranslatePipe, etc.
  ],
  // NOTE: Only add schemas: [CUSTOM_ELEMENTS_SCHEMA] if your template uses
  // web components or third-party elements that Angular cannot recognize.
  // Avoid it for standard Angular components.
  templateUrl: './{{page-name}}.component.html',
  styleUrl: './{{page-name}}.component.scss',
})
export class {{PageName}}Component implements OnInit {
  // ─── Dependency Injection ──────────────────────────────────────────────────
  // Inject ONLY state services in page components.
  // Never inject use cases, repositories, or HttpClient directly.
  private readonly state = inject({{ModuleName}}StateService);

  // ─── State Signals (derived from state service) ────────────────────────────
  // Expose state to template via computed signals.
  // This keeps the template declarative and avoids direct state service exposure.
  readonly items = computed(() => this.state.items());
  readonly loading = computed(() => this.state.loading());
  readonly error = computed(() => this.state.error());
  readonly totalItems = computed(() => this.state.totalItems());
  readonly currentPage = computed(() => this.state.currentPage());
  readonly pageSize = computed(() => this.state.pageSize());

  // Derived booleans — convenient for template conditionals
  readonly hasData = computed(() => this.items().length > 0);
  readonly isEmpty = computed(() => !this.loading() && this.items().length === 0);

  // ─── Local UI State ────────────────────────────────────────────────────────
  // Use signals for local UI state that doesn't belong in the state service
  readonly showCreateModal = signal(false);
  readonly selectedItem = signal<{{EntityName}} | null>(null);

  // ─── Search ────────────────────────────────────────────────────────────────
  private readonly searchSubject = new Subject<string>();

  constructor() {
    // Wire up debounced search — takeUntilDestroyed() handles cleanup automatically
    this.searchSubject.pipe(
      debounceTime(300),
      takeUntilDestroyed(),
    ).subscribe(term => this.state.search(term));

    // Use effect() sparingly — only for side effects that cannot be reactive.
    // Good use case: logging errors, triggering analytics, etc.
    // Do NOT use effect() to update other signals (causes circular dependencies).
    effect(() => {
      const err = this.error();
      if (err) {
        console.error('[{{PageName}}Component] Error:', err);
        // Show toast notification if your project uses one:
        // this.toastService.error(err);
      }
    });
  }

  // ─── Lifecycle ─────────────────────────────────────────────────────────────
  ngOnInit(): void {
    // Trigger initial data load
    this.state.load();
  }

  // ─── Event Handlers ────────────────────────────────────────────────────────

  onSearch(term: string): void {
    this.searchSubject.next(term);
  }

  onPageChange(event: { page: number; pageSize: number }): void {
    this.state.setPage(event.page);
  }

  onRowClick(item: {{EntityName}}): void {
    // Option A: Navigate to detail route
    // this.router.navigate([item.id], { relativeTo: this.route });

    // Option B: Open a detail panel or modal
    this.selectedItem.set(item);
  }

  onCreate(): void {
    this.selectedItem.set(null);
    this.showCreateModal.set(true);
  }

  onEdit(item: {{EntityName}}): void {
    this.selectedItem.set(item);
    this.showCreateModal.set(true);
  }

  onDelete(item: {{EntityName}}): void {
    // Option A: Immediate delete
    this.state.delete(item.id);

    // Option B: Confirm before delete
    // if (await this.confirmDialog.confirm('Delete this item?')) {
    //   this.state.delete(item.id);
    // }
  }

  onModalClosed(result: {{EntityName}} | null): void {
    this.showCreateModal.set(false);
    this.selectedItem.set(null);
    if (result) {
      // Optionally reload or let state handle optimistic update
      this.state.load();
    }
  }

  // ─── Track Function ────────────────────────────────────────────────────────
  // Always provide a track function for @for loops to improve DOM recycling.
  trackById(index: number, item: {{EntityName}}): string {
    return item.id;
  }
}
```

## HTML Template — `{{page-name}}.component.html`

```html
<!-- Page wrapper -->
<div class="page-container">

  <!-- Header -->
  <div class="page-header">
    <h1 class="page-title">
      <!-- Your page title text or translated string -->
      {{PageName}}
    </h1>
    <div class="page-actions">
      <!-- Replace with your library's button component -->
      <button (click)="onCreate()">
        Add New
      </button>
    </div>
  </div>

  <!-- Search / Filters -->
  <div class="page-filters">
    <!-- Replace with your library's search input component -->
    <input
      type="search"
      placeholder="Search..."
      (input)="onSearch($any($event.target).value)"
    />
  </div>

  <!-- Error State -->
  @if (error()) {
    <div class="error-banner">
      {{ error() }}
      <button (click)="state.load()">Retry</button>
    </div>
  }

  <!-- Data Grid -->
  <!-- The grid component is a pure presentational child component -->
  <app-{{entity-name}}-grid
    [data]="items()"
    [loading]="loading()"
    [totalItems]="totalItems()"
    [currentPage]="currentPage()"
    [pageSize]="pageSize()"
    (rowClick)="onRowClick($event)"
    (pageChange)="onPageChange($event)"
    (edit)="onEdit($event)"
    (delete)="onDelete($event)"
    (add)="onCreate()"
  />

  <!-- Create / Edit Modal -->
  <!-- Replace with your library's modal/dialog/drawer component -->
  @if (showCreateModal()) {
    <!-- Your modal wrapper here -->
    <app-{{entity-name}}-form
      [mode]="selectedItem() ? 'edit' : 'create'"
      [entity]="selectedItem()"
      (saved)="onModalClosed($event)"
      (cancelled)="onModalClosed(null)"
    />
  }

</div>
```

## Route Configuration

```typescript
// {{module-name}}.routes.ts
import { Routes } from '@angular/router';
import { {{PageName}}Component } from './presentation/pages/{{page-name}}.component';
import { {{ModuleName}}InfrastructureProviders } from './infrastructure/providers';
import { {{ModuleName}}PresentationProviders } from './presentation/providers';

export const {{ModuleName}}Routes: Routes = [
  {
    path: '',
    component: {{PageName}}Component,
    providers: [
      // Infrastructure providers supply the repository implementations
      ...{{ModuleName}}InfrastructureProviders,
      // Presentation providers supply the state service
      ...{{ModuleName}}PresentationProviders,
    ],
  },
];
```

## Checklist

- [ ] `standalone: true`
- [ ] `ChangeDetectionStrategy.OnPush`
- [ ] Injects only state services — no use cases, repos, or HttpClient
- [ ] Exposes state via `computed()` signals — no direct state service access in template
- [ ] `searchSubject` + `debounceTime` + `takeUntilDestroyed` for search
- [ ] `ngOnInit` calls `state.load()`
- [ ] All event handlers are thin — delegate to state service
- [ ] `trackById` function provided
- [ ] `@if`, `@for`, `@switch` — no `*ngIf`, `*ngFor`
- [ ] Routes configured with infrastructure and presentation providers
- [ ] Component exported from `presentation/index.ts`
