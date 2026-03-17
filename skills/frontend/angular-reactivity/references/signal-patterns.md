# Signal Patterns

## State Service Pattern

The standard pattern for signal-based state services:

```typescript
import { DestroyRef, Injectable, computed, inject, signal } from '@angular/core';
import { takeUntilDestroyed, toObservable } from '@angular/core/rxjs-interop';
import { combineLatest, debounceTime, switchMap } from 'rxjs';

@Injectable()
export class EntityListStateService {
  private readonly destroyRef = inject(DestroyRef);
  private readonly query = inject(GetEntitiesQuery);
  private readonly deleteUseCase = inject(DeleteEntityUseCase);
  private readonly notification = inject(NotificationService);

  // ===== Filter State (private writable signals) =====
  private readonly _keyword = signal('');
  private readonly _page = signal(1);
  private readonly _pageSize = signal(20);
  private readonly _orderBy = signal<string | undefined>(undefined);
  private readonly _orderDirection = signal<'asc' | 'desc' | undefined>(undefined);

  // ===== Data State =====
  private readonly _items = signal<readonly Entity[]>([]);
  private readonly _total = signal(0);
  private readonly _loading = signal(false);
  private readonly _error = signal<string | null>(null);

  // ===== Manual Refresh =====
  private readonly _manualRefresh = signal(0);

  // ===== Public Readonly =====
  readonly items = this._items.asReadonly();
  readonly total = this._total.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly error = this._error.asReadonly();
  readonly keyword = this._keyword.asReadonly();
  readonly page = this._page.asReadonly();
  readonly pageSize = this._pageSize.asReadonly();

  // ===== Computed =====
  readonly totalPages = computed(() => Math.ceil(this._total() / this._pageSize()));
  readonly hasItems = computed(() => this._items().length > 0);

  readonly filterParams = computed(() => ({
    keyword: this._keyword() || undefined,
    page: this._page(),
    pageSize: this._pageSize(),
    orderBy: this._orderBy(),
    orderDirection: this._orderDirection(),
  }));

  // ===== Auto-Reload Stream =====
  constructor() {
    combineLatest([
      toObservable(this._keyword),
      toObservable(this._page),
      toObservable(this._pageSize),
      toObservable(this._orderBy),
      toObservable(this._orderDirection),
      toObservable(this._manualRefresh),
    ])
      .pipe(
        debounceTime(100),
        switchMap(() => this.fetchData()),
        takeUntilDestroyed(this.destroyRef)
      )
      .subscribe();
  }

  // ===== Setters (reset page on filter change) =====
  setKeyword(keyword: string): void {
    this._keyword.set(keyword);
    this._page.set(1);
  }

  setPage(page: number): void { this._page.set(page); }

  setPageSize(size: number): void {
    this._pageSize.set(size);
    this._page.set(1);
  }

  setSort(orderBy: string | undefined, direction: 'asc' | 'desc' | undefined): void {
    this._orderBy.set(orderBy);
    this._orderDirection.set(direction);
    this._page.set(1);
  }

  refresh(): void { this._manualRefresh.update(v => v + 1); }

  // ===== Data Fetching =====
  private async fetchData(): Promise<void> {
    this._loading.set(true);
    this._error.set(null);
    try {
      const result = await this.query.execute(this.filterParams());
      if (result.isSuccess && result.value) {
        this._items.set(result.value.data);
        this._total.set(result.value.total);
      } else {
        this._error.set(result.errorMessage ?? 'An unknown error occurred');
      }
    } catch (e: unknown) {
      this._error.set(e instanceof Error ? e.message : 'An unknown error occurred');
    } finally {
      this._loading.set(false);
    }
  }

  // ===== Mutation Methods =====
  async deleteItem(entity: Entity): Promise<void> {
    const result = await this.deleteUseCase.execute({ id: entity.id });
    if (result.isSuccess) {
      this._items.update(items => items.filter(item => item.id !== entity.id));
      this._total.update(total => Math.max(0, total - 1));
      this.refresh();
      if (result.successMessage) this.notification.success(result.successMessage);
    } else {
      if (result.errorMessage) this.notification.error(result.errorMessage);
    }
  }
}
```

## Rules

- `@Injectable()` only — NO `providedIn: 'root'` (provided at component level)
- Private `_signals` exposed via `.asReadonly()` public signals
- `combineLatest` + `toObservable` + `debounceTime(100)` + `switchMap` for auto-reload
- Always inject `DestroyRef` explicitly; pass to `takeUntilDestroyed(this.destroyRef)`
