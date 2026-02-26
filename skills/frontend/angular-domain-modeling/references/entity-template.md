# Entity Template

Use this template when creating a new domain entity. Replace `{{EntityName}}` with the PascalCase entity name and `{{entityName}}` with the camelCase form.

## File Location

```
<module-path>/src/lib/domain/entities/{{entity-name}}.entity.ts
```

## Template

```typescript
import { Result } from '@your-org/core';

// ---------------------------------------------------------------------------
// Branded ID
// ---------------------------------------------------------------------------

export type {{EntityName}}Id = string & { readonly __brand: '{{EntityName}}Id' };

export const create{{EntityName}}Id = (raw: string): {{EntityName}}Id => {
  if (!raw || raw.trim() === '') {
    throw new Error('{{EntityName}}Id cannot be empty');
  }
  return raw as {{EntityName}}Id;
};

// ---------------------------------------------------------------------------
// Entity Interface
// ---------------------------------------------------------------------------

export interface {{EntityName}} {
  readonly id: {{EntityName}}Id;
  readonly name: string;
  // Add your project-specific context fields here (e.g., tenantId, createdBy)
  readonly createdAt: Date;
  readonly updatedAt: Date;
}

// ---------------------------------------------------------------------------
// Factory
// ---------------------------------------------------------------------------

export interface Create{{EntityName}}Props {
  id: string;
  name: string;
  // Add your project-specific context fields here
}

export const create{{EntityName}} = (
  props: Create{{EntityName}}Props
): Result<{{EntityName}}> => {
  if (!props.id || props.id.trim() === '') {
    return Result.failure<{{EntityName}}>('{{EntityName}} id is required');
  }

  if (!props.name || props.name.trim() === '') {
    return Result.failure<{{EntityName}}>('{{EntityName}} name is required');
  }

  const now = new Date();

  const entity: {{EntityName}} = {
    id: create{{EntityName}}Id(props.id),
    name: props.name.trim(),
    // Map your project-specific context fields here
    createdAt: now,
    updatedAt: now,
  };

  return Result.success(entity);
};

// ---------------------------------------------------------------------------
// Equality
// ---------------------------------------------------------------------------

export const equals{{EntityName}} = (
  a: {{EntityName}},
  b: {{EntityName}}
): boolean => a.id === b.id;

// ---------------------------------------------------------------------------
// Pure Update Function
// ---------------------------------------------------------------------------

export interface Update{{EntityName}}Props {
  name?: string;
  // Add updatable fields here
}

export const update{{EntityName}} = (
  entity: {{EntityName}},
  changes: Update{{EntityName}}Props
): Result<{{EntityName}}> => {
  const name = changes.name !== undefined ? changes.name.trim() : entity.name;

  if (name === '') {
    return Result.failure<{{EntityName}}>('{{EntityName}} name cannot be empty');
  }

  const updated: {{EntityName}} = {
    ...entity,
    name,
    // Apply other field changes here
    updatedAt: new Date(),
  };

  return Result.success(updated);
};
```

## Notes

- All properties must be `readonly`.
- The factory function is the only way to construct a valid entity — never use object literals directly in application code.
- `update{{EntityName}}` returns a **new object**; it never mutates the original.
- Add domain-specific validation rules inside `create{{EntityName}}` and `update{{EntityName}}`.
- Do not add class methods. All logic lives in standalone exported functions.
