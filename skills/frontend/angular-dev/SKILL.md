---
name: angular-dev
description: Angular 17+ development guide covering Standalone Components, Signals-based state, NgRx, RxJS patterns, and integration with .NET/C# backends. Use when user asks to "create Angular component", "implement signal store", "setup NgRx", "write Angular service", "call .NET API from Angular", "handle HTTP in Angular", "fix RxJS subscription leak", "create Angular feature module", "implement lazy routing", "setup interceptor", or needs Angular architecture advice. Always use this skill for any Angular coding task, even if the user doesn't say "Angular" explicitly but the context is clearly frontend + .NET backend integration.
---

# Angular Dev

Angular 17+ development guide — Standalone Components, Signals, NgRx, and .NET backend integration.

## Stack assumptions
- Angular **17+** — Standalone Components (no NgModules by default)
- State: **Signals** for local/simple state, **NgRx Signal Store** for feature state
- HTTP: `HttpClient` with typed responses, interceptors for auth
- Backend: **.NET Web API** (RESTful, likely following KiotViet CQRS pattern)
- Styling: SCSS, standalone imports
- Testing: Jest (or Jasmine) + Angular Testing Library

---

## Quick reference — when to read which file

| Task | Read |
|------|------|
| Build components, templates, directives | `references/component-patterns.md` |
| NgRx Signal Store, Signals, state design | `references/state-management.md` |
| HTTP service, interceptors, .NET integration | `references/backend-integration.md` |

---

## Core principles

### 1. Standalone-first
Always generate standalone components unless user explicitly requests NgModule.
```typescript
@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [CommonModule, RouterModule],
  templateUrl: './product-list.component.html',
})
export class ProductListComponent { }
```

### 2. Signals for local state
Use `signal()` and `computed()` instead of class properties + manual change detection.
```typescript
export class CartComponent {
  items = signal<CartItem[]>([]);
  total = computed(() => this.items().reduce((s, i) => s + i.price, 0));

  addItem(item: CartItem) {
    this.items.update(current => [...current, item]);
  }
}
```

### 3. Typed HTTP — always
Every API call must have a typed response interface matching the .NET DTO.
```typescript
// ✅ typed
this.http.get<ProductDto[]>('/api/products')

// ❌ avoid
this.http.get<any>('/api/products')
```

### 4. Inject function over constructor DI
```typescript
// ✅ Angular 17+ style
export class ProductService {
  private http = inject(HttpClient);
  private store = inject(ProductStore);
}
```

---

## Feature structure (lazy-loaded)

```
src/app/features/products/
├── products.routes.ts         ← Lazy route definition
├── components/
│   ├── product-list/
│   │   ├── product-list.component.ts
│   │   ├── product-list.component.html
│   │   └── product-list.component.scss
│   └── product-detail/
├── services/
│   └── product.service.ts     ← HTTP calls
├── store/
│   └── product.store.ts       ← NgRx Signal Store
└── models/
    └── product.model.ts       ← DTOs + interfaces
```

**Lazy route setup:**
```typescript
// products.routes.ts
export const PRODUCT_ROUTES: Routes = [
  { path: '', component: ProductListComponent },
  { path: ':id', component: ProductDetailComponent },
];

// app.routes.ts
{
  path: 'products',
  loadChildren: () =>
    import('./features/products/products.routes').then(m => m.PRODUCT_ROUTES),
}
```

---

## .NET backend integration — quick patterns

### Service base URL
```typescript
// environment.ts
export const environment = {
  apiUrl: 'https://api.kiotviet.com/v1'
};
```

### HTTP service pattern
```typescript
@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private baseUrl = `${environment.apiUrl}/products`;

  getAll(query: ProductQuery): Observable<PagedResult<ProductDto>> {
    const params = new HttpParams({ fromObject: { ...query } });
    return this.http.get<PagedResult<ProductDto>>(this.baseUrl, { params });
  }

  getById(id: number): Observable<ProductDto> {
    return this.http.get<ProductDto>(`${this.baseUrl}/${id}`);
  }

  create(command: CreateProductCommand): Observable<number> {
    return this.http.post<number>(this.baseUrl, command);
  }

  update(id: number, command: UpdateProductCommand): Observable<void> {
    return this.http.put<void>(`${this.baseUrl}/${id}`, command);
  }
}
```

### Auth interceptor (JWT)
```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  if (!token) return next(req);

  return next(req.clone({
    setHeaders: { Authorization: `Bearer ${token}` }
  }));
};
```

Register in `app.config.ts`:
```typescript
provideHttpClient(withInterceptors([authInterceptor]))
```

---

## Error handling pattern

Match .NET `Result<T>` / ProblemDetails response:
```typescript
export interface ApiError {
  title: string;
  status: number;
  errors?: Record<string, string[]>;
}

// In service
getProduct(id: number): Observable<ProductDto> {
  return this.http.get<ProductDto>(`${this.baseUrl}/${id}`).pipe(
    catchError((err: HttpErrorResponse) => {
      const apiError = err.error as ApiError;
      return throwError(() => apiError);
    })
  );
}
```

---

## State management decision tree

```
Is state shared across multiple components/routes?
├── YES → Use NgRx Signal Store  (see references/state-management.md)
└── NO  → Is it async (HTTP)?
          ├── YES → signal() + service call in component
          └── NO  → signal() only
```

---

## Common anti-patterns to avoid

| ❌ Anti-pattern | ✅ Fix |
|-----------------|--------|
| `subscribe()` without `takeUntilDestroyed()` | Always add `takeUntilDestroyed(this.destroyRef)` |
| `any` typed HTTP responses | Define DTO interfaces matching .NET response |
| Business logic in component | Move to service or store |
| `NgModule` for new features | Use Standalone + lazy routes |
| Manual `ChangeDetectorRef.markForCheck()` | Use Signals + `OnPush` |

---

## References

- **Component patterns & templates** → `references/component-patterns.md`
- **NgRx Signal Store & Signals** → `references/state-management.md`
- **HTTP, interceptors, .NET integration** → `references/backend-integration.md`
