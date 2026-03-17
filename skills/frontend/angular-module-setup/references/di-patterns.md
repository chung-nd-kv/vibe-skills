# DI Patterns

## Full Infrastructure Providers Example

```typescript
import { Provider } from '@angular/core';

// Domain tokens
import { ENTITY_REPOSITORY } from '../../domain';

// Application - Queries
import { GetEntitiesQuery } from '../../application/queries';

// Application - UseCases
import { CreateEntityUseCase, UpdateEntityUseCase, DeleteEntityUseCase } from '../../application/usecases';

// Infrastructure
import { EntityApiClient } from '../api/entity-api.client';
import { EntityRepositoryImpl } from '../repositories/entity.repository';

export const EntityInfrastructureProviders: Provider[] = [
  // Queries
  GetEntitiesQuery,

  // Use Cases
  CreateEntityUseCase,
  UpdateEntityUseCase,
  DeleteEntityUseCase,

  // API Clients
  EntityApiClient,

  // Repository Implementations
  { provide: ENTITY_REPOSITORY, useClass: EntityRepositoryImpl },
];
```

## Repository Token Pattern

```typescript
// domain/entity.repository.ts
export interface EntityRepository {
  find(params: FilterParams): Promise<Result<PagedResult<Entity>>>;
  create(params: CreateParams): Promise<Result<Entity>>;
  update(params: UpdateParams): Promise<Result<Entity>>;
  delete(id: EntityId): Promise<Result<void>>;
}

export const ENTITY_REPOSITORY = new InjectionToken<EntityRepository>('EntityRepository');

// infrastructure/repositories/entity.repository.ts
@Injectable()
export class EntityRepositoryImpl implements EntityRepository {
  private readonly apiClient = inject(EntityApiClient);
  // ... implementation
}
```

## Route Configuration with Providers

```typescript
export const entityRoutes: Routes = [
  {
    path: '',
    loadComponent: () => import('./pages/entity-page.component').then(m => m.EntityPageComponent),
    providers: [...EntityInfrastructureProviders],
  },
];
```

## Provider Scope Comparison

| Scope | When | Lifetime |
|-------|------|----------|
| Route-level `providers` | Infrastructure (repos, queries, usecases) | Route active |
| Component `providers` | State services scoped to page | Component active |
| `providedIn: 'root'` | Global singletons (auth, config) | App lifetime |
