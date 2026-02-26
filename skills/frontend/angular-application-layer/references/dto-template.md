# DTO Template

Data Transfer Objects (DTOs) define the shape of data exchanged between the application and external systems (typically a backend API). They are plain TypeScript interfaces — no classes, no decorators, no runtime overhead.

## PascalCase Convention

Properties use **PascalCase** by default to match typical API contracts. This is configurable — if your API uses camelCase or snake_case, adjust accordingly and update the mapper to translate between conventions.

## File Location

```
<module-path>/src/lib/application/dtos/{{entityName}}.dto.ts
```

---

## Template

```typescript
// ============================================
// Response DTO (from API)
// ============================================

/**
 * DTO for {{EntityName}} response from API.
 * Maps directly to API response structure.
 *
 * Note: Property names use PascalCase to match API contract.
 * Adjust to camelCase or snake_case if your API differs.
 */
export interface {{EntityName}}Dto {
  readonly Id: number;
  readonly Name: string;
  readonly Description?: string;
  // Add your entity-specific fields here
  readonly CreatedDate?: string;  // ISO date string from API
  readonly ModifiedDate?: string;
  readonly CreatedBy: number;
  readonly ModifiedBy?: number;
  readonly Position?: number;
  readonly IsActive: boolean;
}

// ============================================
// Create DTO (to API)
// ============================================

/**
 * DTO for creating a new {{EntityName}}.
 * Properties use PascalCase to match API contract.
 */
export interface Create{{EntityName}}Dto {
  /** Required: Display name */
  readonly Name: string;

  /** Optional: Description */
  readonly Description?: string;

  // Add other fields required/optional for creation
  // Note: Fields like CreatedBy, CreatedDate are typically
  // set server-side and should NOT be included here
}

// ============================================
// Update DTO (to API)
// ============================================

/**
 * DTO for updating an existing {{EntityName}}.
 * Id is required to identify the entity.
 */
export interface Update{{EntityName}}Dto {
  /** Required: Entity ID */
  readonly Id: number;

  /** Required: Updated name */
  readonly Name: string;

  /** Optional: Updated description */
  readonly Description?: string;

  /** Optional: Updated display order */
  readonly Position?: number;

  /** Optional: Updated active status */
  readonly IsActive?: boolean;

  // Add other updatable fields here
}

// ============================================
// API Response Wrappers
// ============================================

/**
 * API response wrapper for list endpoints.
 *
 * Adjust the shape to match your backend's envelope format.
 * Some APIs use { data: [] }, others use { items: [] }, etc.
 */
export interface {{EntityName}}ListResponseDto {
  Data: {{EntityName}}Dto[];
}

/**
 * API response wrapper for single entity endpoints.
 */
export interface {{EntityName}}ResponseDto {
  Message?: string;
  Data: {{EntityName}}Dto;
}

/**
 * Generic API message response for mutations (create/update/delete).
 * Use when the API returns only a status message without entity data.
 */
export interface ApiMessageResponseDto {
  Message?: string;
}
```

---

## Barrel Export

Create `<module-path>/src/lib/application/dtos/index.ts`:

```typescript
export * from './{{entityName}}.dto';
```

---

## Usage Notes

- **DTOs are read-only** — use `readonly` on all properties to prevent accidental mutation
- **Optional fields** — use `?` for fields that may be absent in the API response
- **Date fields** — always type as `string` in DTOs (ISO 8601 format); the Mapper converts to `Date`
- **ID fields** — always type as `number` or `string` (primitive); the Mapper wraps in branded types
- **Nullable vs optional** — `field?: string` (optional, may be absent) vs `field: string | null` (present but null); match your API contract exactly
