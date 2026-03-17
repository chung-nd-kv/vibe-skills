---
name: angular-hexagonal
description: Hexagonal/clean architecture for Angular with domain layer (branded IDs, readonly interfaces, pure functions), application layer (QueryBase, UseCaseBase, DTOs, Mappers), infrastructure layer (API clients, repositories), and presentation layer. Use when structuring modules, understanding layer boundaries, working with Result<T>, branded types, or creating files in domain/application/infrastructure/presentation directories.
---

# Hexagonal Architecture for Angular

Four-layer architecture: Domain → Application → Infrastructure → Presentation.

## Layer Dependency Rules

| Layer          | Can Depend On       | Contains                                                  |
| -------------- | ------------------- | --------------------------------------------------------- |
| Domain         | Nothing             | Branded IDs, interfaces, pure functions, repository ports |
| Application    | Domain              | QueryBase, UseCaseBase, DTOs, Mappers                     |
| Infrastructure | Domain, Application | API clients, repository impls, providers                  |
| Presentation   | Application         | Pages, components, state services, routes                 |

## Domain Layer — Plain Interfaces + Functions

```typescript
// Branded Types for type-safe IDs
export type EntityId = number & { __brand: 'EntityId' };
export const EntityId = (value: number) => value as EntityId;

// Readonly Interface
export interface Entity {
  readonly id: EntityId;
  readonly name: string;
  readonly isActive: boolean;
}

// Factory Function → Result<T>
export function createEntity(params: CreateEntityParams): Result<Entity> {
  if (!params.name?.trim()) return Result.failure('Name is required');
  return Result.success({ id: EntityId(0), name: params.name.trim(), isActive: true });
}

// Pure Functions
export function canDeactivate(entity: Entity): boolean {
  return entity.isActive;
}
```

## Application Layer — Deep Nesting

```
application/
├── queries/<entity>/get-<entity>s/get-<entity>s.query.ts
├── usecases/<entity>/create-<entity>/create-<entity>.usecase.ts
├── dtos/<entity>.dto.ts
└── mappers/<entity>.mapper.ts
```

### QueryBase (reads) / UseCaseBase (writes)

```typescript
@Injectable()
export class GetEntitiesQuery extends QueryBase<FilterParams, PagedResult<Entity>> {
  private readonly repository = inject(ENTITY_REPOSITORY);

  async execute(params: FilterParams): Promise<Result<PagedResult<Entity>>> {
    return this.repository.find(params);
  }
}
```

## Result<T> Pattern

```typescript
Result.success(value);
Result.success(value, message);
Result.failure<T>(errorMessage);
result.isSuccess / result.isFailure;
result.value;
result.errorMessage / result.successMessage;
```

## Mapper — Static Class

```typescript
export class EntityMapper {
  static toDomain(dto: EntityDto): Entity { ... }
  static toCreateDto(params: CreateEntityParams): CreateEntityRequest { ... }
}
```

Never `@Injectable()`. No state. No DI.

## Module File Structure

```
libs/features/<feature>/src/lib/
├── domain/
│   ├── entities/<entity>.ts
│   ├── repositories/<entity>.repository.ts
│   └── index.ts
├── application/
│   ├── queries/<entity>/get-<entity>s/
│   ├── usecases/<entity>/create-<entity>/
│   ├── dtos/<entity>.dto.ts
│   ├── mappers/<entity>.mapper.ts
│   └── index.ts
├── infrastructure/
│   ├── api/<entity>-api.client.ts
│   ├── repositories/<entity>.repository.ts
│   ├── providers/index.ts
│   └── index.ts
└── presentation/
    ├── pages/<entity>/
    ├── components/<entity>-form/
    ├── state/<entity>-list-state.service.ts
    ├── <feature>.routes.ts
    └── index.ts
```

## Barrel Exports

Every directory has an `index.ts` that re-exports its contents.

For detailed layer patterns, see [references/layer-patterns.md](references/layer-patterns.md).
For code generation workflow, see [references/code-generation.md](references/code-generation.md).
