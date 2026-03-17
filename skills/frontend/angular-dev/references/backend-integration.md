# Backend Integration — Angular + .NET Web API

## DTO mapping conventions

Map .NET response DTOs 1-to-1 in TypeScript. Follow naming from the .NET side.

```typescript
// models/product.model.ts

// Mirrors .NET ProductDto
export interface ProductDto {
  id: number;
  code: string;
  name: string;
  price: number;
  discountPercent: number;
  categoryId: number;
  isActive: boolean;
  createdAt: string;   // ISO date string from .NET
}

// Mirrors .NET CreateProductCommand
export interface CreateProductCommand {
  code: string;
  name: string;
  price: number;
  categoryId: number;
}

// Mirrors .NET UpdateProductCommand
export interface UpdateProductCommand {
  name: string;
  price: number;
  categoryId: number;
}

// Mirrors .NET paged result (common CQRS pattern)
export interface PagedResult<T> {
  items: T[];
  totalCount: number;
  page: number;
  pageSize: number;
}

// Mirrors .NET ProductQuery
export interface ProductFilter {
  page: number;
  pageSize: number;
  search?: string;
  categoryId?: number;
  isActive?: boolean;
}
```

---

## HTTP service — full CRUD pattern

```typescript
// services/product.service.ts
@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private baseUrl = `${environment.apiUrl}/products`;

  getAll(filter: ProductFilter): Observable<PagedResult<ProductDto>> {
    const params = new HttpParams({ fromObject: this.toHttpParams(filter) });
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

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }

  // Convert filter object → query params (skip null/undefined)
  private toHttpParams(obj: Record<string, unknown>): Record<string, string> {
    return Object.fromEntries(
      Object.entries(obj)
        .filter(([, v]) => v !== null && v !== undefined)
        .map(([k, v]) => [k, String(v)])
    );
  }
}
```

---

## Interceptors

### Auth interceptor (JWT Bearer)
```typescript
// interceptors/auth.interceptor.ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authStore = inject(AuthStore);
  const token = authStore.token();

  if (!token) return next(req);

  return next(req.clone({
    setHeaders: { Authorization: `Bearer ${token}` }
  }));
};
```

### Error interceptor (global handler)
```typescript
// interceptors/error.interceptor.ts
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      switch (error.status) {
        case 401:
          router.navigate(['/login']);
          break;
        case 403:
          router.navigate(['/forbidden']);
          break;
        case 404:
          // let component handle
          break;
        case 422:
          // validation errors — let component handle via throwError
          break;
        case 500:
          // global toast / log
          console.error('Server error', error);
          break;
      }
      return throwError(() => error);
    })
  );
};
```

### Register in app.config.ts
```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(appRoutes),
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    ),
  ],
};
```

---

## Handling .NET validation errors (ProblemDetails)

.NET returns `422 Unprocessable Entity` with `ProblemDetails`:
```json
{
  "title": "Validation failed",
  "status": 422,
  "errors": {
    "Name": ["Name is required"],
    "Price": ["Price must be greater than 0"]
  }
}
```

Map to Angular form errors:
```typescript
export interface ProblemDetails {
  title: string;
  status: number;
  errors?: Record<string, string[]>;
}

// In component after submit
submit() {
  this.productService.create(this.form.getRawValue()).subscribe({
    next: (id) => this.router.navigate(['/products', id]),
    error: (err: HttpErrorResponse) => {
      if (err.status === 422) {
        const problem = err.error as ProblemDetails;
        this.applyServerErrors(problem.errors ?? {});
      }
    }
  });
}

private applyServerErrors(errors: Record<string, string[]>) {
  Object.entries(errors).forEach(([field, messages]) => {
    const control = this.form.get(field.toLowerCase());
    control?.setErrors({ serverError: messages[0] });
  });
}
```

Template:
```html
<mat-error *ngIf="nameControl.hasError('serverError')">
  {{ nameControl.getError('serverError') }}
</mat-error>
```

---

## Environment config

```typescript
// environments/environment.ts  (dev)
export const environment = {
  production: false,
  apiUrl: 'http://localhost:5000/api',
};

// environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.kiotviet.com/api',
};
```

Angular 17+ uses `app.config.ts` for environment — no `environment.ts` file injection needed, just import directly.

---

## CORS note (for .NET side)

Remind team that .NET must allow Angular dev origin:
```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("Angular", policy =>
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod());
});
app.UseCors("Angular");
```

---

## Checklist — new HTTP service

- [ ] DTO interfaces defined in `models/` folder
- [ ] Service uses `inject(HttpClient)` (not constructor DI)
- [ ] All methods return typed `Observable<T>`
- [ ] `toHttpParams()` helper skips null/undefined
- [ ] Auth interceptor registered in `app.config.ts`
- [ ] Error interceptor handles 401/403 globally
- [ ] 422 validation errors mapped back to form controls
