---
name: angular-http
description: Angular HTTP client patterns with HttpClient, two-layer architecture (API Client returns Observable, Repository wraps with Result), error handling, and DTO conventions. Use when creating API clients, handling HTTP requests, or implementing repository adapters.
---

# Angular HTTP Patterns

Two-layer pattern: API Client (Observable) → Repository (Result<T>).

## API Client Layer

Returns `Observable<DTO>` — no Result wrapping here:

```typescript
import { HttpClient, HttpParams } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable, catchError, throwError } from 'rxjs';

@Injectable()
export class EntityApiClient {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = inject(API_BASE_URL);

  findAll(params?: HttpParams): Observable<EntityListResponse> {
    return this.http.get<EntityListResponse>(
      `${this.baseUrl}/entities`, { params }
    ).pipe(
      catchError(error => throwError(() => new Error(this.extractError(error))))
    );
  }

  create(payload: CreateEntityRequest): Observable<EntityResponse> {
    return this.http.post<EntityResponse>(
      `${this.baseUrl}/entities`, payload
    ).pipe(
      catchError(error => throwError(() => new Error(this.extractError(error))))
    );
  }

  private extractError(error: unknown): string {
    if (error instanceof Error) return error.message;
    if (typeof error === 'object' && error !== null && 'error' in error) {
      const httpError = error as { error?: { message?: string } };
      return httpError.error?.message ?? 'Request failed';
    }
    return 'Unknown error';
  }
}
```

## Repository Layer

Wraps API calls with `Result<T>` using `firstValueFrom`:

```typescript
import { firstValueFrom } from 'rxjs';

@Injectable()
export class EntityRepositoryImpl implements EntityRepository {
  private readonly apiClient = inject(EntityApiClient);

  async find(params: FilterParams): Promise<Result<PagedResult<Entity>>> {
    try {
      const response = await firstValueFrom(this.apiClient.findAll(this.buildParams(params)));
      return Result.success({
        data: response.data.map(EntityMapper.toDomain),
        total: response.total,
      });
    } catch (error: unknown) {
      return Result.failure(error instanceof Error ? error.message : 'Unknown error');
    }
  }

  async create(params: CreateEntityParams): Promise<Result<Entity>> {
    try {
      const response = await firstValueFrom(
        this.apiClient.create(EntityMapper.toCreateDto(params))
      );
      return Result.success(EntityMapper.toDomain(response.data));
    } catch (error: unknown) {
      return Result.failure(error instanceof Error ? error.message : 'Failed to create');
    }
  }
}
```

## Key Rules

- API Client returns `Observable<DTO>`, NOT `Result<T>`
- Repository uses `firstValueFrom()` to convert Observable → Promise
- Every API client method has `catchError` with error extraction
- All repository methods return `Promise<Result<T>>`
- Never expose raw HTTP errors to consumers

For detailed patterns, see [references/http-patterns.md](references/http-patterns.md).
