# Layer Patterns

## Domain File Structure

Each domain file follows this order:

```typescript
// 1. Branded Types
export type EntityId = number & { __brand: 'EntityId' };
export const EntityId = (value: number) => value as EntityId;

// 2. Status/Enum Types
export type EntityStatus = 'active' | 'inactive';

// 3. Readonly Interface
export interface Entity {
  readonly id: EntityId;
  readonly name: string;
  readonly description?: string;
  readonly isActive: boolean;
  readonly createdAt: Date;
}

// 4. Create/Update Params
export interface CreateEntityParams {
  name: string;
  description?: string;
}

export interface UpdateEntityParams extends CreateEntityParams {
  id: EntityId;
}

// 5. Factory Function with Validation
export function createEntity(params: CreateEntityParams): Result<Entity> {
  if (!params.name?.trim()) return Result.failure('Name is required');
  if (params.name.length > 255) return Result.failure('Name must be 255 characters or less');
  return Result.success({
    id: EntityId(0),
    name: params.name.trim(),
    description: params.description,
    isActive: true,
    createdAt: new Date(),
  });
}

// 6. Pure Functions
export function canDeactivate(entity: Entity): boolean {
  return entity.isActive;
}
```

## Repository Interface (Port)

```typescript
import { InjectionToken } from '@angular/core';

export interface EntityRepository {
  find(params: EntityFilterParams): Promise<Result<PagedResult<Entity>>>;
  findById(id: EntityId): Promise<Result<Entity | null>>;
  create(params: CreateEntityParams): Promise<Result<Entity>>;
  update(params: UpdateEntityParams): Promise<Result<Entity>>;
  delete(id: EntityId): Promise<Result<void>>;
}

export const ENTITY_REPOSITORY = new InjectionToken<EntityRepository>('EntityRepository');
```

## Application Deep Nesting

```
application/
├── queries/
│   ├── index.ts
│   └── entity/
│       ├── index.ts
│       └── get-entities/
│           ├── get-entities.query.ts
│           └── index.ts
├── usecases/
│   ├── index.ts
│   └── entity/
│       ├── index.ts
│       ├── create-entity/
│       ├── update-entity/
│       └── delete-entity/
├── dtos/
│   └── entity.dto.ts
└── mappers/
    └── entity.mapper.ts
```

Each `index.ts` re-exports from its children.

## DTO Conventions

DTOs match API format:

```typescript
export interface EntityDto {
  readonly id: number;
  readonly name: string;
  readonly isActive: boolean;
  readonly createdAt: string;
}

export interface EntityListResponseDto {
  readonly data: EntityDto[];
  readonly total: number;
}
```

## Mapper Pattern

Static class with `toDomain` and `toDto` methods:

```typescript
export class EntityMapper {
  static toDomain(dto: EntityDto): Entity {
    return {
      id: EntityId(dto.id),
      name: dto.name,
      isActive: dto.isActive,
      createdAt: new Date(dto.createdAt),
    };
  }

  static toCreateDto(params: CreateEntityParams): CreateEntityRequest {
    return { name: params.name, description: params.description };
  }
}
```
