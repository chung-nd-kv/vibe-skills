# State Service Template

This template provides the full Angular Signals-based state service for the presentation layer. Replace `{{EntityName}}` with the PascalCase entity name and `{{entityName}}` with the camelCase entity name throughout.

---

## Imports

```typescript
import { Injectable, computed, inject, signal } from '@angular/core';
import { firstValueFrom, race } from 'rxjs';
import { map } from 'rxjs/operators';

// Import your domain entity and ID types from the domain layer
import { {{EntityName}}, {{EntityName}}Id } from '../../domain';

// Import your application-layer use cases and queries
import {
  Create{{EntityName}}UseCase,
  Update{{EntityName}}UseCase,
  Delete{{EntityName}}UseCase,
  Get{{EntityName}}sQuery,
  Get{{EntityName}}Query,
} from '../../application';

// Import DTOs for create/update payloads
import { Create{{EntityName}}Dto, Update{{EntityName}}Dto } from '../../application';

// Import your core Result type
import { Result } from '@your-org/core';

// Import your UI library's modal and toast services here
// Example: import { MatDialog } from '@angular/material/dialog';
// Example: import { ToastService } from '@your-org/ui';

// Import your i18n service here
// Example: import { TranslateService } from '@ngx-translate/core';
```

---

## Full State Service Class

```typescript
@Injectable()
export class {{EntityName}}StateService {

  // ─── Use Cases ────────────────────────────────────────────────────────────

  private readonly get{{EntityName}}sQuery = inject(Get{{EntityName}}sQuery);
  private readonly get{{EntityName}}Query = inject(Get{{EntityName}}Query);
  private readonly create{{EntityName}}UseCase = inject(Create{{EntityName}}UseCase);
  private readonly update{{EntityName}}UseCase = inject(Update{{EntityName}}UseCase);
  private readonly delete{{EntityName}}UseCase = inject(Delete{{EntityName}}UseCase);

  // Inject your UI services here, e.g.:
  // private readonly dialog = inject(MatDialog);
  // private readonly toastService = inject(ToastService);
  // private readonly translate = inject(TranslateService);

  // ─── Private Writable Signals ──────────────────────────────────────────────

  private readonly _items = signal<readonly {{EntityName}}[]>([]);
  private readonly _selectedId = signal<{{EntityName}}Id | null>(null);
  private readonly _loading = signal<boolean>(false);
  private readonly _error = signal<string | null>(null);
  private readonly _hasAnyItem = signal<boolean>(false);

  // ─── Public Readonly Signals ───────────────────────────────────────────────

  readonly items = this._items.asReadonly();
  readonly selectedId = this._selectedId.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();
  readonly hasAnyItem = this._hasAnyItem.asReadonly();

  // ─── Computed Signals ──────────────────────────────────────────────────────

  readonly selectedItem = computed(() => {
    const id = this._selectedId();
    if (id === null) return null;
    return this._items().find(i => i.id === id) ?? null;
  });

  readonly activeItems = computed(() =>
    this._items().filter(i => i.isActive)
  );

  readonly hasItems = computed(() => this._items().length > 0);

  readonly total = computed(() => this._items().length);

  // ─── Load & Refresh ────────────────────────────────────────────────────────

  async load(): Promise<void> {
    this._loading.set(true);
    this._error.set(null);
    try {
      const result = await this.get{{EntityName}}sQuery.execute();
      if (result.isSuccess && result.value) {
        this._items.set(result.value);
        this._hasAnyItem.set(result.value.length > 0);
      } else {
        this._error.set(result.errorMessage ?? 'Failed to load');
      }
    } finally {
      this._loading.set(false);
    }
  }

  async refresh(): Promise<void> {
    return this.load();
  }

  // ─── CRUD Actions ──────────────────────────────────────────────────────────

  async create(dto: Create{{EntityName}}Dto): Promise<boolean> {
    this._loading.set(true);
    this._error.set(null);
    try {
      const result = await this.create{{EntityName}}UseCase.execute(dto);
      if (result.isSuccess) {
        // Show success notification, e.g.:
        // this.toastService.success('Created successfully');
        await this.refresh();
        return true;
      } else {
        this._error.set(result.errorMessage ?? 'Failed to create');
        // Show error notification, e.g.:
        // this.toastService.error(this._error());
        return false;
      }
    } finally {
      this._loading.set(false);
    }
  }

  async update(dto: Update{{EntityName}}Dto): Promise<boolean> {
    this._loading.set(true);
    this._error.set(null);
    try {
      const result = await this.update{{EntityName}}UseCase.execute(dto);
      if (result.isSuccess) {
        // Show success notification, e.g.:
        // this.toastService.success('Updated successfully');
        await this.refresh();
        return true;
      } else {
        this._error.set(result.errorMessage ?? 'Failed to update');
        // Show error notification, e.g.:
        // this.toastService.error(this._error());
        return false;
      }
    } finally {
      this._loading.set(false);
    }
  }

  async delete(id: {{EntityName}}Id): Promise<boolean> {
    this._loading.set(true);
    this._error.set(null);
    try {
      const result = await this.delete{{EntityName}}UseCase.execute({ id });
      if (result.isSuccess) {
        // Show success notification, e.g.:
        // this.toastService.success('Deleted successfully');
        // Deselect if the deleted item was selected
        if (this._selectedId() === id) {
          this._selectedId.set(null);
        }
        await this.refresh();
        return true;
      } else {
        this._error.set(result.errorMessage ?? 'Failed to delete');
        // Show error notification, e.g.:
        // this.toastService.error(this._error());
        return false;
      }
    } finally {
      this._loading.set(false);
    }
  }

  // ─── Modal Methods (Optional) ──────────────────────────────────────────────
  //
  // Use these when the state service is responsible for orchestrating create/edit dialogs.
  // Adapt `modalService.open`, `afterSubmitted()`, and `afterClosed()` to your UI library
  // (Angular Material MatDialog, PrimeNG DynamicDialog, custom overlay service, etc.).

  async openCreateDialog(): Promise<{{EntityName}} | null> {
    // Example pattern — replace with your actual modal service API:
    //
    // const modalRef = this.dialog.open({{EntityName}}FormComponent, {
    //   data: { mode: 'create' },
    // });
    //
    // const result = await firstValueFrom(
    //   race(
    //     modalRef.afterClosed().pipe(filter(Boolean)),
    //     modalRef.afterClosed().pipe(map(() => null))
    //   )
    // );
    //
    // if (result) await this.refresh();
    // return result ?? null;

    throw new Error('openCreateDialog: implement with your modal service');
  }

  async openEditDialog(id: {{EntityName}}Id): Promise<{{EntityName}} | null> {
    const entity = await this.getById(id);
    if (!entity) return null;

    // Example pattern — replace with your actual modal service API:
    //
    // const modalRef = this.dialog.open({{EntityName}}FormComponent, {
    //   data: { mode: 'edit', entity },
    // });
    //
    // const result = await firstValueFrom(
    //   race(
    //     modalRef.afterClosed().pipe(filter(Boolean)),
    //     modalRef.afterClosed().pipe(map(() => null))
    //   )
    // );
    //
    // if (result) await this.refresh();
    // return result ?? null;

    throw new Error('openEditDialog: implement with your modal service');
  }

  // ─── Selection Helpers ─────────────────────────────────────────────────────

  select(id: {{EntityName}}Id | null): void {
    this._selectedId.set(id);
  }

  async getById(id: {{EntityName}}Id): Promise<{{EntityName}} | null> {
    // Check local cache first
    const cached = this._items().find(i => i.id === id);
    if (cached) return cached;

    // Fetch from remote
    const result = await this.get{{EntityName}}Query.execute({ id });
    if (result.isSuccess && result.value) {
      return result.value;
    }
    return null;
  }

  clearError(): void {
    this._error.set(null);
  }
}
```

---

## Provider Registration

```typescript
// <module-path>/src/lib/presentation/presentation.providers.ts
import { Provider } from '@angular/core';
import { {{EntityName}}StateService } from './state/{{entityName}}.state.service';

export const presentationProviders: Provider[] = [
  {{EntityName}}StateService,
  // Add other presentation-layer services here
];
```

Wire into feature routes — do NOT use `providedIn: 'root'`:

```typescript
// In your route configuration:
{
  path: '{{entityName}}s',
  providers: [...presentationProviders],
  component: {{EntityName}}ShellComponent,
}
```

---

## Usage in Components

Inject the state service and bind signals directly to the template:

```typescript
@Component({
  selector: 'app-{{entityName}}-list',
  template: `
    @if (state.loading()) {
      <app-loading-spinner />
    }

    @if (state.error()) {
      <app-error-banner [message]="state.error()!" (dismiss)="state.clearError()" />
    }

    @for (item of state.items(); track item.id) {
      <app-{{entityName}}-card
        [entity]="item"
        [selected]="item.id === state.selectedId()"
        (select)="state.select(item.id)"
        (edit)="state.openEditDialog(item.id)"
        (delete)="state.delete(item.id)"
      />
    }
  `,
})
export class {{EntityName}}ListComponent {
  readonly state = inject({{EntityName}}StateService);

  ngOnInit(): void {
    this.state.load();
  }
}
```

---

## Notes

- Replace `@your-org/core` with your actual core library path that exports `Result<T>`
- Replace all `{{EntityName}}` placeholders with the actual entity name (e.g., `Product`, `Category`)
- Replace all `{{entityName}}` placeholders with the camelCase entity name (e.g., `product`, `category`)
- The modal methods are optional — remove them if your components handle modal orchestration directly
- `activeItems` computed signal assumes an `isActive` property on the entity — adapt to your domain model
