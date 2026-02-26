# Value Object Template

Use this template when creating a new domain value object. Replace `{{ValueObjectName}}` with the PascalCase name.

Value objects have **no identity**. Two value objects with identical property values are considered equal.

## File Location

```
<module-path>/src/lib/domain/value-objects/{{value-object-name}}.value-object.ts
```

## Template

```typescript
import { Result } from '@your-org/core';

// ---------------------------------------------------------------------------
// Value Object Interface
// ---------------------------------------------------------------------------

export interface {{ValueObjectName}} {
  readonly value: string; // Replace with the actual fields for this value object
  // readonly field2: number;
}

// ---------------------------------------------------------------------------
// Factory
// ---------------------------------------------------------------------------

export interface Create{{ValueObjectName}}Props {
  value: string; // Replace with actual input fields
}

export const create{{ValueObjectName}} = (
  props: Create{{ValueObjectName}}Props
): Result<{{ValueObjectName}}> => {
  if (!props.value || props.value.trim() === '') {
    return Result.failure<{{ValueObjectName}}>(
      '{{ValueObjectName}} value is required'
    );
  }

  // Add format/range/business rule validation here
  // Example:
  // if (props.value.length > 255) {
  //   return Result.failure<{{ValueObjectName}}>('{{ValueObjectName}} exceeds maximum length');
  // }

  const valueObject: {{ValueObjectName}} = {
    value: props.value.trim(),
  };

  return Result.success(valueObject);
};

// ---------------------------------------------------------------------------
// Equality
// ---------------------------------------------------------------------------

export const equals{{ValueObjectName}} = (
  a: {{ValueObjectName}},
  b: {{ValueObjectName}}
): boolean => {
  // Compare all fields — value objects have no identity
  return a.value === b.value;
};

// ---------------------------------------------------------------------------
// Transform / Derived Value Helpers
// ---------------------------------------------------------------------------

/**
 * Returns the primitive representation of the value object.
 * Useful for serialisation or displaying in the UI.
 */
export const {{valueObjectName}}ToString = (vo: {{ValueObjectName}}): string =>
  vo.value;

// Add additional transformation helpers as needed, e.g.:
// export const {{valueObjectName}}ToNumber = (vo: {{ValueObjectName}}): number => Number(vo.value);
```

## Notes

- Value objects must not have an `id` property.
- All properties must be `readonly`.
- Equality is determined by value comparison, not by reference or identity.
- Factory functions must validate all invariants before constructing the object.
- Transform helpers keep conversion logic co-located with the value object definition.
- Do not add class methods. All logic lives in standalone exported functions.
