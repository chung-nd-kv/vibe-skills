# RxJS Event Patterns

## Dialog Pattern

Open a dialog, wait for submit or close, handle the result:

```typescript
import { firstValueFrom, race } from 'rxjs';
import { map } from 'rxjs/operators';

private async handleDelete(entity: Entity): Promise<void> {
  const dialogRef = this.dialog.open(ConfirmDialogComponent, {
    title: 'Delete Confirmation',
    inputs: { entity },
  });

  // race() handles both submit and close without EmptyError
  const confirmed = await firstValueFrom(
    race(
      dialogRef.afterSubmitted(),
      dialogRef.afterClosed().pipe(map(() => null))
    )
  );

  if (confirmed !== true) return;
  await this.state.deleteItem(entity);
}
```

## Nested Dialog Pattern

Open a dialog from inside another dialog:

```typescript
async handleAddGroup(): Promise<void> {
  const dialogRef = this.dialog.open(GroupFormComponent, {
    title: 'Create Group',
    inputs: { mode: 'create' },
  });

  const result = await firstValueFrom(
    race(dialogRef.afterSubmitted(), dialogRef.afterClosed().pipe(map(() => null)))
  );

  if (result) {
    await this.groupState.loadGroups();
    this.form.patchValue({ groupId: result.id });
  }
}
```

## Dialog Service Abstraction

For complex modules, create a dedicated dialog service:

```typescript
@Injectable()
export class EntityDialogService {
  private readonly dialog = inject(DialogService);

  openCreateForm(): Observable<Entity | null> {
    const ref = this.dialog.open(EntityFormComponent, {
      title: 'Create Entity',
      inputs: { mode: 'create' },
    });
    return race(ref.afterSubmitted(), ref.afterClosed().pipe(map(() => null)));
  }

  openEditForm(entity: Entity): Observable<Entity | null> {
    const ref = this.dialog.open(EntityFormComponent, {
      title: 'Edit Entity',
      inputs: { mode: 'edit', entity },
    });
    return race(ref.afterSubmitted(), ref.afterClosed().pipe(map(() => null)));
  }
}
```

Usage in page component:

```typescript
this.addClick$
  .pipe(
    switchMap(() => this.dialogService.openCreateForm()),
    tap(created => { if (created) this.state.refresh(); }),
    takeUntilDestroyed(this.destroyRef)
  )
  .subscribe();
```

## CRUD Stream Pattern

Handle create/edit/delete with auto-refresh:

```typescript
ngOnInit(): void {
  // Add: switchMap allows cancellation
  this.addClick$.pipe(
    switchMap(() => this.dialogService.openCreateForm()),
    tap(created => { if (created) this.state.refresh(); }),
    takeUntilDestroyed(this.destroyRef)
  ).subscribe();

  // Edit: switchMap for cancel-and-restart
  this.editClick$.pipe(
    switchMap(entity => this.dialogService.openEditForm(entity)),
    tap(updated => { if (updated) this.state.refresh(); }),
    takeUntilDestroyed(this.destroyRef)
  ).subscribe();

  // Delete: switchMap + async handler with confirmation
  this.deleteClick$.pipe(
    switchMap(entity => from(this.handleDelete(entity))),
    takeUntilDestroyed(this.destroyRef)
  ).subscribe();
}
```

## Deactivation Confirmation Pattern

Conditional confirmation dialog based on current state:

```typescript
private async handleToggleStatus(entity: Entity): Promise<void> {
  if (entity.isActive) {
    const confirmed = await this.dialog.confirm('Deactivate this entity?');
    if (!confirmed) return;
  }
  await this.state.updateStatus(entity.id, !entity.isActive);
}
```

## The Hybrid Pattern

Signals hold state, RxJS orchestrates changes:

```typescript
// Signals = state
private readonly _items = signal<Entity[]>([]);
private readonly _loading = signal(false);

// RxJS = orchestration
combineLatest([toObservable(this._keyword), toObservable(this._page)])
  .pipe(
    debounceTime(100),
    switchMap(() => this.fetchData()), // fetchData updates signals
    takeUntilDestroyed(this.destroyRef)
  )
  .subscribe();
```
