# Repository Port Template

Use this template when creating a new domain repository port. Replace `{{EntityName}}` with the PascalCase entity name and `{{entityName}}` with the camelCase form.

A repository port is an **interface** that the domain defines and the infrastructure layer implements. The `InjectionToken` is the only Angular import permitted in the domain layer.

## File Location

```
<module-path>/src/lib/domain/ports/{{entity-name}}.repository.port.ts
```

## Template

```typescript
import { InjectionToken } from '@angular/core';
import { Result } from '@your-org/core';
import { {{EntityName}}, {{EntityName}}Id } from '../entities/{{entity-name}}.entity';

// ---------------------------------------------------------------------------
// Filter Params
// ---------------------------------------------------------------------------

/**
 * Generic filter parameters for list queries.
 * Extend with domain-specific filter fields as needed.
 */
export interface {{EntityName}}FilterParams {
  page?: number;
  pageSize?: number;
  search?: string;
  // Add domain-specific filter fields here, e.g.:
  // status?: string;
  // fromDate?: Date;
}

export interface {{EntityName}}ListResult {
  readonly items: {{EntityName}}[];
  readonly total: number;
  readonly page: number;
  readonly pageSize: number;
}

// ---------------------------------------------------------------------------
// Repository Port Interface
// ---------------------------------------------------------------------------

export interface {{EntityName}}RepositoryPort {
  /**
   * Returns a paginated list of entities matching the given filter.
   */
  findAll(
    filter?: {{EntityName}}FilterParams
  ): Promise<Result<{{EntityName}}ListResult>>;

  /**
   * Returns a single entity by its ID, or failure if not found.
   */
  findById(id: {{EntityName}}Id): Promise<Result<{{EntityName}}>>;

  /**
   * Persists a new entity and returns the saved entity.
   */
  create(entity: {{EntityName}}): Promise<Result<{{EntityName}}>>;

  /**
   * Updates an existing entity and returns the updated entity.
   */
  update(entity: {{EntityName}}): Promise<Result<{{EntityName}}>>;

  /**
   * Deletes an entity by its ID.
   */
  delete(id: {{EntityName}}Id): Promise<Result<void>>;

  /**
   * Returns the total number of entities matching the given filter.
   */
  count(filter?: {{EntityName}}FilterParams): Promise<Result<number>>;
}

// ---------------------------------------------------------------------------
// Injection Token
// ---------------------------------------------------------------------------

export const {{EntityName_UPPER}}_REPOSITORY_PORT =
  new InjectionToken<{{EntityName}}RepositoryPort>(
    '{{EntityName}}RepositoryPort'
  );
```

> Replace `{{EntityName_UPPER}}` with the SCREAMING_SNAKE_CASE form of the entity name, e.g., `PRODUCT_REPOSITORY_PORT`.

## Registering the Token

In your infrastructure module or `providers` array, bind the token to the concrete implementation:

```typescript
import { {{EntityName_UPPER}}_REPOSITORY_PORT } from '../domain/ports/{{entity-name}}.repository.port';
import { {{EntityName}}HttpRepository } from './repositories/{{entity-name}}.http-repository';

// In your @NgModule providers or ApplicationConfig providers:
{
  provide: {{EntityName_UPPER}}_REPOSITORY_PORT,
  useClass: {{EntityName}}HttpRepository,
}
```

## Injecting the Port in Application Layer

```typescript
import { inject } from '@angular/core';
import { {{EntityName_UPPER}}_REPOSITORY_PORT } from '../../domain/ports/{{entity-name}}.repository.port';

export class Get{{EntityName}}UseCase {
  private readonly repository = inject({{EntityName_UPPER}}_REPOSITORY_PORT);

  async execute(id: string): Promise<Result<{{EntityName}}>> {
    const entityId = create{{EntityName}}Id(id);
    return this.repository.findById(entityId);
  }
}
```

## Notes

- `InjectionToken` is the only Angular import permitted in the domain layer.
- Never use abstract classes for repository ports — interfaces compose better and carry no runtime overhead.
- All methods return `Promise<Result<T>>` to make success and failure explicit without exceptions.
- Keep the port interface minimal; add methods only when the domain requires them.
- The infrastructure implementation (e.g., `{{EntityName}}HttpRepository`) lives outside the domain and maps API DTOs to domain entities.
