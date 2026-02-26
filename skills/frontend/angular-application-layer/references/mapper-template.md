# Mapper Template

Mappers translate between API DTOs and domain entities. They are pure transformation classes with static methods — stateless, no side effects, no DI dependencies.

## File Location

```
<module-path>/src/lib/application/mappers/{{entityName}}.mapper.ts
```

---

## Template

```typescript
import { Result } from '@your-org/core';
import {
  {{EntityName}},
  create{{EntityName}},
  {{EntityName}}Id,
  // Import other branded types your entity uses, for example:
  // UserId,
  // TenantId,
} from '../../domain';
import { {{EntityName}}Dto } from '../dtos/{{entityName}}.dto';

/**
 * Mapper for converting between {{EntityName}} DTOs and domain entities.
 *
 * Uses static methods for stateless transformation.
 * Handles branded types, date parsing, and validation.
 *
 * @example
 * ```typescript
 * // DTO to Domain
 * const entity = {{EntityName}}Mapper.toDomain(dto);
 *
 * // Domain to DTO
 * const dto = {{EntityName}}Mapper.toDto(entity);
 *
 * // List mapping
 * const entities = {{EntityName}}Mapper.toDomainList(dtos);
 * ```
 */
export class {{EntityName}}Mapper {
  /**
   * Convert API DTO to domain entity.
   *
   * Handles:
   * - Branded types (wraps primitive IDs into branded types)
   * - Date parsing (ISO string → Date)
   * - Validation via domain factory function
   *
   * @param dto - The API response DTO
   * @returns Domain entity
   * @throws Error if DTO fails domain validation
   */
  static toDomain(dto: {{EntityName}}Dto): {{EntityName}} {
    const result = create{{EntityName}}({
      id: {{EntityName}}Id(dto.Id),
      name: dto.Name,
      description: dto.Description,
      // Map your entity-specific fields here.
      // Wrap primitive IDs in their branded types, for example:
      // createdBy: UserId(dto.CreatedBy),
      // tenantId: TenantId(dto.TenantId),
      createdDate: dto.CreatedDate ? new Date(dto.CreatedDate) : undefined,
      modifiedDate: dto.ModifiedDate ? new Date(dto.ModifiedDate) : undefined,
      createdBy: dto.CreatedBy,
      modifiedBy: dto.ModifiedBy,
      position: dto.Position,
      isActive: dto.IsActive ?? false,
    });

    if (!result.isSuccess) {
      throw new Error(
        `Failed to map {{EntityName}}Dto to domain: ${result.errorMessage}`
      );
    }

    return result.value as {{EntityName}};
  }

  /**
   * Convert domain entity to API DTO.
   *
   * Strips branded types to primitives.
   * Converts Date to ISO string.
   *
   * @param entity - The domain entity
   * @returns API DTO
   */
  static toDto(entity: {{EntityName}}): {{EntityName}}Dto {
    return {
      Id: entity.id,
      Name: entity.name,
      Description: entity.description,
      // Map your entity-specific fields back to primitives here.
      // Branded types are structurally compatible with their underlying
      // primitive type, so direct assignment works:
      // CreatedBy: entity.createdBy,
      // TenantId: entity.tenantId,
      CreatedDate: entity.createdDate?.toISOString(),
      ModifiedDate: entity.modifiedDate?.toISOString(),
      CreatedBy: entity.createdBy,
      ModifiedBy: entity.modifiedBy,
      Position: entity.position,
      IsActive: entity.isActive,
    };
  }

  /**
   * Convert array of DTOs to domain entities.
   *
   * @param dtos - Array of API DTOs
   * @returns Array of domain entities
   */
  static toDomainList(dtos: {{EntityName}}Dto[]): {{EntityName}}[] {
    return dtos.map(dto => {{EntityName}}Mapper.toDomain(dto));
  }

  /**
   * Convert array of domain entities to DTOs.
   *
   * @param entities - Array of domain entities
   * @returns Array of API DTOs
   */
  static toDtoList(entities: {{EntityName}}[]): {{EntityName}}Dto[] {
    return entities.map(entity => {{EntityName}}Mapper.toDto(entity));
  }
}
```

---

## Barrel Export

Create `<module-path>/src/lib/application/mappers/index.ts`:

```typescript
export * from './{{entityName}}.mapper';
```

---

## Design Notes

### Why Static Methods?

Mappers are stateless transformations. There is no state to manage and no dependencies to inject, so static methods are the appropriate choice. They are easier to call (`{{EntityName}}Mapper.toDomain(dto)`) and easier to test (no class instantiation needed).

### Branded Types

Domain entities use branded types (e.g., `{{EntityName}}Id`, `UserId`) to prevent accidentally mixing up IDs of different entity types. The mapper is the boundary where:

- **`toDomain`** wraps incoming primitive values into branded types using type constructor functions
- **`toDto`** unwraps branded types back to primitives (TypeScript's structural typing handles this automatically)

Add your project-specific branded type imports and mappings where the comments indicate.

### Date Handling

- API sends dates as ISO 8601 strings (e.g., `"2024-01-15T10:30:00Z"`)
- Domain entities use JavaScript `Date` objects
- `toDomain` parses strings to `Date` with a null guard
- `toDto` serializes `Date` to ISO string with `toISOString()`

### Error Handling in toDomain

`toDomain` throws an `Error` if the domain factory returns a failure. This is intentional — a DTO that cannot be mapped to a valid domain entity indicates a data contract violation. The error will propagate to the repository implementation, which should handle it appropriately.
