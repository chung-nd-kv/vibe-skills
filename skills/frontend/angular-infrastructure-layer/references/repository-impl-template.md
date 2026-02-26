# Repository Implementation Template

A concrete repository implementation that implements the domain port interface using an API client and a mapper. All methods return `Result<T>` — a discriminated union — so callers never deal with raw exceptions.

---

## Prerequisites

- Domain port interface and its `InjectionToken` must exist in the domain layer.
- A DTO and mapper must exist in the application layer.
- The API client for the entity must exist in the infrastructure layer.

---

## Full Template

```typescript
// File: <module-path>/src/lib/infrastructure/repositories/{{entityName}}.repository.ts

import { Injectable, inject } from '@angular/core';
import { Result } from '@your-org/core'; // replace with your Result type import

import { {{EntityName}} } from '../../domain/entities/{{entityName}}';
import { {{EntityName}}Repository } from '../../domain/ports/{{entityName}}-repository.interface';
import { {{EntityName}}Mapper } from '../../application/mappers/{{entityName}}.mapper';
import { {{EntityName}}ApiClient } from '../api/{{entityName}}-api.client';
import { Create{{EntityName}}Dto } from '../../application/dtos/create-{{entityName}}.dto';
import { Update{{EntityName}}Dto } from '../../application/dtos/update-{{entityName}}.dto';

export interface Find{{EntityName}}sOptions {
  page?: number;
  pageSize?: number;
  search?: string;
}

export interface Paginated{{EntityName}}s {
  items: {{EntityName}}[];
  total: number;
  page: number;
  pageSize: number;
}

@Injectable()
export class {{EntityName}}RepositoryImpl implements {{EntityName}}Repository {
  private readonly apiClient = inject({{EntityName}}ApiClient);
  private readonly mapper = inject({{EntityName}}Mapper);

  async findAll(options?: Find{{EntityName}}sOptions): Promise<Result<Paginated{{EntityName}}s>> {
    try {
      const response = await this.apiClient.findAll(options);
      return Result.success({
        items: response.items.map(dto => this.mapper.toDomain(dto)),
        total: response.total,
        page: response.page,
        pageSize: response.pageSize,
      });
    } catch (error) {
      return Result.failure(extractRepoError(error));
    }
  }

  async findById(id: string): Promise<Result<{{EntityName}}>> {
    try {
      const dto = await this.apiClient.findById(id);
      return Result.success(this.mapper.toDomain(dto));
    } catch (error) {
      return Result.failure(extractRepoError(error));
    }
  }

  async create(data: Create{{EntityName}}Dto): Promise<Result<{{EntityName}}>> {
    try {
      const dto = await this.apiClient.create(data);
      return Result.success(this.mapper.toDomain(dto));
    } catch (error) {
      return Result.failure(extractRepoError(error));
    }
  }

  async update(id: string, data: Update{{EntityName}}Dto): Promise<Result<{{EntityName}}>> {
    try {
      const dto = await this.apiClient.update(id, data);
      return Result.success(this.mapper.toDomain(dto));
    } catch (error) {
      return Result.failure(extractRepoError(error));
    }
  }

  async delete(id: string): Promise<Result<void>> {
    try {
      await this.apiClient.delete(id);
      return Result.success(undefined);
    } catch (error) {
      return Result.failure(extractRepoError(error));
    }
  }

  async count(options?: Find{{EntityName}}sOptions): Promise<Result<number>> {
    try {
      const response = await this.apiClient.findAll({ ...options, pageSize: 1 });
      return Result.success(response.total);
    } catch (error) {
      return Result.failure(extractRepoError(error));
    }
  }
}

function extractRepoError(error: unknown): string {
  if (error instanceof Error) return error.message;
  if (typeof error === 'object' && error !== null && 'message' in error) {
    return String((error as any).message);
  }
  return 'An unexpected error occurred';
}
```

---

## Result Type

`Result<T>` is a discriminated union with two variants:

```typescript
// Typical Result type pattern (define once in a shared/core package)
export type Result<T> =
  | { success: true; value: T }
  | { success: false; error: string };

export const Result = {
  success: <T>(value: T): Result<T> => ({ success: true, value }),
  failure: <T>(error: string): Result<T> => ({ success: false, error }),
};
```

Replace `@your-org/core` with the actual import path used in your project.

---

## Template Variables

| Variable | Description | Example |
|---|---|---|
| `{{EntityName}}` | PascalCase entity name | `Product`, `Order`, `Customer` |
| `{{entityName}}` | camelCase entity name | `product`, `order`, `customer` |

---

## Notes

- The `count` method re-uses `findAll` with `pageSize: 1` to read `total`. Replace with a dedicated API endpoint if one exists.
- The repository must `implement` the domain port interface — this is what enforces the contract.
- `@Injectable()` has no `providedIn` — provisioned via the provider array.
- No `HttpClient` usage here; all HTTP logic lives in the API client.
- The mapper is injected (not instantiated directly) so it can be a stateful or mockable service if needed.
