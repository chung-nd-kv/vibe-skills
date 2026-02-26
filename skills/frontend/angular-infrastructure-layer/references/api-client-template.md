# API Client Template

An HTTP API client that wraps Angular's `HttpClient`, converting Observables to Promises and centralizing endpoint logic. This class is consumed by repository implementations — it does not implement domain ports directly.

---

## InjectionToken Definition

Define the base URL token in a shared location (e.g., `infrastructure/tokens/api-base-url.token.ts`):

```typescript
import { InjectionToken } from '@angular/core';

export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');
```

Provide it at the app or route level:

```typescript
// app.config.ts or route providers
{ provide: API_BASE_URL, useValue: environment.apiBaseUrl }
```

---

## Error Helper

Inline this function in the API client file (no shared utility import needed):

```typescript
function extractApiError(error: unknown): string {
  if (error instanceof Error) return error.message;
  if (typeof error === 'object' && error !== null && 'message' in error) {
    return String((error as any).message);
  }
  return 'An unexpected error occurred';
}
```

---

## Full Template

```typescript
// File: <module-path>/src/lib/infrastructure/api/{{entityName}}-api.client.ts

import { Injectable, inject } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { firstValueFrom } from 'rxjs';
import { InjectionToken } from '@angular/core';

import { {{EntityName}}Dto } from '../../application/dtos/{{entityName}}.dto';
import { Create{{EntityName}}Dto } from '../../application/dtos/create-{{entityName}}.dto';
import { Update{{EntityName}}Dto } from '../../application/dtos/update-{{entityName}}.dto';

export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');

function extractApiError(error: unknown): string {
  if (error instanceof Error) return error.message;
  if (typeof error === 'object' && error !== null && 'message' in error) {
    return String((error as any).message);
  }
  return 'An unexpected error occurred';
}

export interface {{EntityName}}QueryParams {
  page?: number;
  pageSize?: number;
  search?: string;
  [key: string]: unknown;
}

export interface Paginated{{EntityName}}Response {
  items: {{EntityName}}Dto[];
  total: number;
  page: number;
  pageSize: number;
}

@Injectable()
export class {{EntityName}}ApiClient {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = inject(API_BASE_URL);
  private readonly endpoint = `${this.baseUrl}/{{apiEndpoint}}`;

  async findAll(params?: {{EntityName}}QueryParams): Promise<Paginated{{EntityName}}Response> {
    const httpParams = this.buildQueryParams(params);
    return firstValueFrom(
      this.http.get<Paginated{{EntityName}}Response>(this.endpoint, { params: httpParams })
    );
  }

  async findById(id: string): Promise<{{EntityName}}Dto> {
    return firstValueFrom(
      this.http.get<{{EntityName}}Dto>(`${this.endpoint}/${id}`)
    );
  }

  async create(dto: Create{{EntityName}}Dto): Promise<{{EntityName}}Dto> {
    return firstValueFrom(
      this.http.post<{{EntityName}}Dto>(this.endpoint, dto)
    );
  }

  async update(id: string, dto: Update{{EntityName}}Dto): Promise<{{EntityName}}Dto> {
    return firstValueFrom(
      this.http.put<{{EntityName}}Dto>(`${this.endpoint}/${id}`, dto)
    );
  }

  async delete(id: string): Promise<void> {
    return firstValueFrom(
      this.http.delete<void>(`${this.endpoint}/${id}`)
    );
  }

  private buildQueryParams(params?: {{EntityName}}QueryParams): HttpParams {
    let httpParams = new HttpParams();
    if (!params) return httpParams;

    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined && value !== null && value !== '') {
        httpParams = httpParams.set(key, String(value));
      }
    });

    return httpParams;
  }
}
```

---

## Template Variables

| Variable | Description | Example |
|---|---|---|
| `{{EntityName}}` | PascalCase entity name | `Product`, `Order`, `Customer` |
| `{{entityName}}` | camelCase entity name | `product`, `order`, `customer` |
| `{{apiEndpoint}}` | API endpoint path segment | `products`, `orders`, `customers` |

---

## Notes

- `@Injectable()` has no `providedIn` — this client is provisioned through the feature's provider array.
- Import DTOs from `../../application/dtos/` (relative path from the infrastructure directory).
- If the API returns a flat list instead of paginated, replace `Paginated{{EntityName}}Response` with `{{EntityName}}Dto[]`.
- The `extractApiError` function is intentionally inlined to avoid infrastructure coupling to a shared utility.
