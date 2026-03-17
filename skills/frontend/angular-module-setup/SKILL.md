---
name: angular-module-setup
description: Angular module setup with inject() function, InjectionToken for repository ports, route-level providers for DI scoping, lazy loading with loadComponent/loadChildren, infrastructure providers array, and functional guards. Use when configuring DI, creating tokens, wiring providers, setting up routes, or understanding injection scope.
---

# Angular Module Setup — DI + Routing

## inject() Function

All DI uses `inject()` function. Never constructor injection.

```typescript
private readonly http = inject(HttpClient);
private readonly destroyRef = inject(DestroyRef);
private readonly repository = inject(ENTITY_REPOSITORY);
private readonly dialogControl = inject(DialogControl, { optional: true });
```

## InjectionToken for Repository Ports

Domain layer defines the port + token:

```typescript
import { InjectionToken } from '@angular/core';

export interface EntityRepository {
  find(params: FilterParams): Promise<Result<PagedResult<Entity>>>;
  create(params: CreateParams): Promise<Result<Entity>>;
  update(params: UpdateParams): Promise<Result<Entity>>;
  delete(id: EntityId): Promise<Result<void>>;
}

export const ENTITY_REPOSITORY = new InjectionToken<EntityRepository>('EntityRepository');
```

## Route-Level Providers

Providers registered at route level, NOT component or root:

```typescript
export const entityRoutes: Routes = [
  {
    path: '',
    loadComponent: () => import('./pages/entity-page.component').then(m => m.EntityPageComponent),
    providers: [...EntityInfrastructureProviders],
  },
];
```

## Infrastructure Providers Array

```typescript
export const EntityInfrastructureProviders: Provider[] = [
  // Queries (Application Layer)
  GetEntitiesQuery,
  // Use Cases (Application Layer)
  CreateEntityUseCase, UpdateEntityUseCase, DeleteEntityUseCase,
  // API Clients (Infrastructure Layer)
  EntityApiClient,
  // Repository Implementations (Adapters)
  { provide: ENTITY_REPOSITORY, useClass: EntityRepositoryImpl },
];
```

**Order:** Queries → UseCases → API Clients → Repository implementations

## Lazy Loading

```typescript
// Single component
{ path: 'entities', loadComponent: () => import('@app/entity').then(m => m.EntityPageComponent) }

// Child routes with shared providers
{ path: 'entities', loadChildren: () => import('@app/entity').then(m => m.entityRoutes) }
```

## Route Params as Signal Inputs

```typescript
// app.config.ts
provideRouter(routes, withComponentInputBinding());

// component
entityId = input<string>(); // auto-bound from :entityId route param
```

## Functional Guards

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  return auth.isAuthenticated() || inject(Router).createUrlTree(['/login']);
};
```

## DestroyRef — Always Inject Explicitly

```typescript
private readonly destroyRef = inject(DestroyRef);

// In ngOnInit — MUST pass destroyRef
stream$.pipe(takeUntilDestroyed(this.destroyRef)).subscribe();
```

## State Services — Component-Level Providers

```typescript
@Component({
  providers: [EntityListStateService],
})
export class EntityPageComponent {
  readonly state = inject(EntityListStateService);
}
```

For detailed patterns, see [references/di-patterns.md](references/di-patterns.md).
