# HTTP Patterns

## Full API Client Example

```typescript
import { HttpClient, HttpParams } from '@angular/common/http';
import { Injectable, InjectionToken, inject } from '@angular/core';
import { Observable, catchError, throwError } from 'rxjs';

export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');

@Injectable()
export class EntityApiClient {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = inject(API_BASE_URL);
  private readonly endpoint = '/api/entities';

  findAll(params?: HttpParams): Observable<EntityListResponseDto> {
    return this.http.get<EntityListResponseDto>(
      `${this.baseUrl}${this.endpoint}`, { params }
    ).pipe(
      catchError(error => throwError(() => new Error(extractErrorMessage(error))))
    );
  }

  findById(id: number): Observable<EntityResponseDto> {
    return this.http.get<EntityResponseDto>(
      `${this.baseUrl}${this.endpoint}/${id}`
    ).pipe(
      catchError(error => throwError(() => new Error(extractErrorMessage(error))))
    );
  }

  create(payload: CreateEntityRequest): Observable<EntityMutationResponseDto> {
    return this.http.post<EntityMutationResponseDto>(
      `${this.baseUrl}${this.endpoint}`, payload
    ).pipe(
      catchError(error => throwError(() => new Error(extractErrorMessage(error))))
    );
  }

  update(id: number, payload: UpdateEntityRequest): Observable<EntityMutationResponseDto> {
    return this.http.put<EntityMutationResponseDto>(
      `${this.baseUrl}${this.endpoint}/${id}`, payload
    ).pipe(
      catchError(error => throwError(() => new Error(extractErrorMessage(error))))
    );
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(
      `${this.baseUrl}${this.endpoint}/${id}`
    ).pipe(
      catchError(error => throwError(() => new Error(extractErrorMessage(error))))
    );
  }
}

// Generic error extraction
function extractErrorMessage(error: unknown): string {
  if (error instanceof Error) return error.message;
  if (typeof error === 'object' && error !== null) {
    const e = error as Record<string, unknown>;
    if (e['error'] && typeof e['error'] === 'object') {
      const inner = e['error'] as Record<string, unknown>;
      if (typeof inner['message'] === 'string') return inner['message'];
    }
    if (typeof e['message'] === 'string') return e['message'];
  }
  return 'Unknown error';
}
```

## Full Repository Implementation

```typescript
import { inject, Injectable } from '@angular/core';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class EntityRepositoryImpl implements EntityRepository {
  private readonly apiClient = inject(EntityApiClient);

  async find(params: EntityFilterParams): Promise<Result<PagedResult<Entity>>> {
    try {
      const httpParams = this.buildHttpParams(params);
      const response = await firstValueFrom(this.apiClient.findAll(httpParams));
      return Result.success({
        data: response.data.map(EntityMapper.toDomain),
        total: response.total,
        page: params.page,
        pageSize: params.pageSize,
      });
    } catch (error: unknown) {
      return Result.failure(error instanceof Error ? error.message : 'Unknown error');
    }
  }

  async create(params: CreateEntityParams): Promise<Result<Entity>> {
    try {
      const dto = EntityMapper.toCreateDto(params);
      const response = await firstValueFrom(this.apiClient.create(dto));
      return Result.success(EntityMapper.toDomain(response.data), response.message);
    } catch (error: unknown) {
      return Result.failure(error instanceof Error ? error.message : 'Failed to create');
    }
  }

  async update(params: UpdateEntityParams): Promise<Result<Entity>> {
    try {
      const dto = EntityMapper.toUpdateDto(params);
      const response = await firstValueFrom(this.apiClient.update(params.id as number, dto));
      return Result.success(EntityMapper.toDomain(response.data), response.message);
    } catch (error: unknown) {
      return Result.failure(error instanceof Error ? error.message : 'Failed to update');
    }
  }

  async delete(id: EntityId): Promise<Result<void>> {
    try {
      await firstValueFrom(this.apiClient.delete(id as number));
      return Result.success(undefined);
    } catch (error: unknown) {
      return Result.failure(error instanceof Error ? error.message : 'Failed to delete');
    }
  }

  private buildHttpParams(params: EntityFilterParams): HttpParams {
    let httpParams = new HttpParams()
      .set('page', params.page.toString())
      .set('pageSize', params.pageSize.toString());

    if (params.keyword) httpParams = httpParams.set('search', params.keyword);
    if (params.orderBy) httpParams = httpParams.set('orderBy', params.orderBy);
    if (params.orderDirection) httpParams = httpParams.set('orderDir', params.orderDirection);

    return httpParams;
  }
}
```

## DTO Conventions

DTOs match the API response format:

```typescript
export interface EntityDto {
  readonly id: number;
  readonly name: string;
  readonly description?: string;
  readonly isActive: boolean;
  readonly createdAt: string;
}

export interface EntityListResponseDto {
  readonly data: readonly EntityDto[];
  readonly total: number;
}

export interface EntityMutationResponseDto {
  readonly data: EntityDto;
  readonly message: string;
}
```

## Common Mistakes

1. Missing `catchError` in API client → unhandled HTTP errors
2. Using `HttpClient` directly in repository → bypasses error handling layer
3. Not using `firstValueFrom` → mixing Observable and Promise patterns
4. Exposing raw HTTP error objects to UI → poor error messages
