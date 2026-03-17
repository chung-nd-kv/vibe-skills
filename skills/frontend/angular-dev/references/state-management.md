# State Management — Signals & NgRx Signal Store

## Decision: Signal vs NgRx Signal Store

| Scenario | Use |
|----------|-----|
| Local UI state (toggle, form dirty) | `signal()` in component |
| Async data in single component | `signal()` + `toSignal()` |
| State shared across multiple components | NgRx Signal Store |
| Complex state with side effects (pagination, filters) | NgRx Signal Store |
| Global app state (auth, cart, user prefs) | NgRx Signal Store |

---

## NgRx Signal Store — standard pattern

```typescript
// product.store.ts
import { signalStore, withState, withComputed, withMethods, patchState } from '@ngrx/signals';
import { withEntities, setEntities, addEntity, updateEntity, removeEntity } from '@ngrx/signals/entities';

export interface ProductState {
  loading: boolean;
  error: string | null;
  filter: ProductFilter;
  selectedId: number | null;
}

const initialState: ProductState = {
  loading: false,
  error: null,
  filter: { page: 1, pageSize: 20 },
  selectedId: null,
};

export const ProductStore = signalStore(
  { providedIn: 'root' },       // or provide in feature route

  withState(initialState),
  withEntities<ProductDto>(),   // normalized entity collection

  withComputed(({ entities, selectedId, filter }) => ({
    selectedProduct: computed(() =>
      entities().find(p => p.id === selectedId()) ?? null
    ),
    totalCount: computed(() => entities().length),
    pagedProducts: computed(() => {
      const { page, pageSize } = filter();
      return entities().slice((page - 1) * pageSize, page * pageSize);
    }),
  })),

  withMethods((store, productService = inject(ProductService)) => ({
    async loadProducts(): Promise<void> {
      patchState(store, { loading: true, error: null });
      try {
        const result = await firstValueFrom(
          productService.getAll(store.filter())
        );
        patchState(store, setEntities(result.items));
      } catch (err) {
        patchState(store, { error: 'Failed to load products' });
      } finally {
        patchState(store, { loading: false });
      }
    },

    async createProduct(command: CreateProductCommand): Promise<void> {
      patchState(store, { loading: true });
      try {
        const id = await firstValueFrom(productService.create(command));
        patchState(store, addEntity({ id, ...command } as ProductDto));
      } finally {
        patchState(store, { loading: false });
      }
    },

    selectProduct(id: number): void {
      patchState(store, { selectedId: id });
    },

    updateFilter(partial: Partial<ProductFilter>): void {
      patchState(store, state => ({
        filter: { ...state.filter, ...partial }
      }));
    },
  }))
);
```

### Provide in feature route (not root)
```typescript
// products.routes.ts
export const PRODUCT_ROUTES: Routes = [
  {
    path: '',
    providers: [ProductStore],  // scoped to this feature
    children: [
      { path: '', component: ProductListComponent },
    ]
  }
];
```

### Consume in component
```typescript
export class ProductListComponent {
  store = inject(ProductStore);

  ngOnInit() {
    this.store.loadProducts();
  }
}
```
```html
@if (store.loading()) {
  <app-spinner />
}

@for (product of store.pagedProducts(); track product.id) {
  <app-product-card
    [product]="product"
    (addToCart)="onAdd($event)"
  />
}

@if (store.error()) {
  <app-error [message]="store.error()!" />
}
```

---

## Signals — local async pattern

For components that don't need shared state:

```typescript
export class ProductDetailComponent {
  private route = inject(ActivatedRoute);
  private productService = inject(ProductService);

  // derive id from route
  private id$ = this.route.paramMap.pipe(
    map(p => Number(p.get('id')))
  );

  // toSignal converts Observable → Signal automatically
  product = toSignal(
    this.id$.pipe(switchMap(id => this.productService.getById(id))),
    { initialValue: null }
  );
}
```

---

## Auth state (global store example)

```typescript
// auth.store.ts
export const AuthStore = signalStore(
  { providedIn: 'root' },
  withState({
    user: null as UserDto | null,
    token: null as string | null,
    loading: false,
  }),
  withComputed(({ user, token }) => ({
    isAuthenticated: computed(() => !!token()),
    userName: computed(() => user()?.fullName ?? ''),
  })),
  withMethods((store, authService = inject(AuthService)) => ({
    async login(credentials: LoginCommand): Promise<void> {
      patchState(store, { loading: true });
      try {
        const result = await firstValueFrom(authService.login(credentials));
        patchState(store, { user: result.user, token: result.token });
      } finally {
        patchState(store, { loading: false });
      }
    },
    logout(): void {
      patchState(store, { user: null, token: null });
    },
  }))
);
```

---

## Common NgRx Signal Store mistakes

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Mutating state directly | Use `patchState()` always |
| Calling `patchState` inside `computed` | Only in `withMethods` |
| Subscribing inside store methods | Use `firstValueFrom()` / `lastValueFrom()` |
| Providing at root when only 1 feature needs it | Provide in feature route |
| Forgetting `withEntities` for list data | Always use for entity collections |
