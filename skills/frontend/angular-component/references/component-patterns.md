# Component Patterns

## Page Component Structure

Page components orchestrate state and user actions:

```typescript
@Component({
  selector: 'app-entity-page',
  imports: [CommonModule, EntityGridComponent, EntityFormComponent],
  templateUrl: './entity-page.component.html',
  styleUrl: './entity-page.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
  providers: [EntityListStateService],
  host: { class: 'page-content' },
})
export class EntityPageComponent implements OnInit {
  // 1. Injected services
  private readonly state = inject(EntityListStateService);
  private readonly destroyRef = inject(DestroyRef);
  private readonly dialog = inject(DialogService);

  // 2. Subject streams for async actions
  private readonly addClick$ = new Subject<void>();
  private readonly deleteClick$ = new Subject<Entity>();

  // 3. Computed state exposure
  readonly tableData = computed(() => [...this.state.items()]);
  readonly loading = this.state.loading;

  // 4. ngOnInit: wire Subject streams
  ngOnInit(): void {
    this.addClick$.pipe(
      switchMap(() => from(this.handleAdd())),
      takeUntilDestroyed(this.destroyRef)
    ).subscribe();
  }

  // 5. Template event handlers (emit to Subject)
  onAdd(): void { this.addClick$.next(); }

  // 6. Private async handlers
  private async handleAdd(): Promise<void> { ... }
}
```

Note: Providers are configured at the **route level** in `*.routes.ts`, not in the component's `providers` array (except state services scoped to a page).

## State Exposure Pattern

Page components expose state service signals via computed wrappers:

```typescript
private readonly state = inject(EntityListStateService);

// Expose as computed (creates new array ref for OnPush)
readonly tableData = computed(() => [...this.state.items()]);

// Direct readonly signal forwarding
readonly loading = this.state.loading;
readonly error = this.state.error;
readonly totalItems = this.state.total;
readonly currentPage = this.state.page;
readonly pageSize = this.state.pageSize;
```

## Dialog Pattern (generic)

```typescript
private async handleAdd(): Promise<void> {
  const result = await this.dialog.open(EntityFormComponent, {
    title: 'Create Entity',
    inputs: { mode: 'create' },
  });

  if (result) {
    this.state.refresh();
  }
}
```
