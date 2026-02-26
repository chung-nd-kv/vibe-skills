# Command Use Case Template

Command use cases handle write operations (create, update, delete). Each use case is a single-responsibility class that executes one business command, validates input through the domain layer, and persists changes via the repository port.

## File Location

```
<module-path>/src/lib/application/usecases/{{entityName}}/
├── create-{{entityName}}.usecase.ts
├── update-{{entityName}}.usecase.ts
├── delete-{{entityName}}.usecase.ts
└── index.ts
```

---

## Execution Context (Optional)

If your use cases need the current user, tenant, or branch context, define an injection token in your core library:

```typescript
// Define in your core library:
export const APP_CONTEXT = new InjectionToken<AppContext>('AppContext');

export interface AppContext {
  userId: number;
  tenantId: number;
  // Add whatever your application needs
}
```

Inject it in use cases that need it:

```typescript
private readonly context = inject(APP_CONTEXT);
```

If your application does not need execution context, omit this injection entirely.

---

## Template

```typescript
import { inject, Injectable } from '@angular/core';
import { Result } from '@your-org/core';
// Optional: import APP_CONTEXT if your use cases need user/tenant context
// import { APP_CONTEXT } from '@your-org/core';
import {
  create{{EntityName}},
  {{EntityName}},
  {{EntityName}}Id,
  {{EntityName}}RepositoryToken,
} from '../../../domain';
import { Create{{EntityName}}Dto, Update{{EntityName}}Dto } from '../../dtos/{{entityName}}.dto';

// ============================================
// Create Use Case
// ============================================

/**
 * Command use case to create a new {{EntityName}}.
 *
 * Handles validation, entity creation, and persistence.
 *
 * @example
 * ```typescript
 * const useCase = inject(Create{{EntityName}}UseCase);
 * const result = await useCase.execute({
 *   Name: 'New Item',
 *   Description: 'Description',
 * });
 *
 * if (result.isSuccess) {
 *   console.log('Created:', result.value);
 * }
 * ```
 */
@Injectable()
export class Create{{EntityName}}UseCase {
  private readonly repository = inject({{EntityName}}RepositoryToken);
  // Inject APP_CONTEXT if needed for user/tenant information:
  // private readonly context = inject(APP_CONTEXT);

  async execute(request: Create{{EntityName}}Dto): Promise<Result<{{EntityName}}>> {
    // Create domain entity using factory function.
    // The factory validates all invariants and returns Result.
    const entityResult = create{{EntityName}}({
      id: {{EntityName}}Id(0), // Server assigns the real ID
      name: request.Name,
      description: request.Description,
      // Set context-derived fields if using APP_CONTEXT:
      // createdBy: UserId(this.context.userId),
      // tenantId: TenantId(this.context.tenantId),
      isActive: true,
    });

    // Return early if domain validation failed
    if (!entityResult.isSuccess) {
      return Result.failure<{{EntityName}}>(
        entityResult.errorMessage ?? 'Unknown validation error'
      );
    }

    // Repository.create() returns Result<{{EntityName}}>
    return this.repository.create(entityResult.value!);
  }
}

// ============================================
// Update Use Case
// ============================================

/**
 * Command use case to update an existing {{EntityName}}.
 *
 * Fetches current entity, applies changes, validates, and persists.
 *
 * @example
 * ```typescript
 * const useCase = inject(Update{{EntityName}}UseCase);
 * const result = await useCase.execute({
 *   Id: 1,
 *   Name: 'Updated Name',
 * });
 * ```
 */
@Injectable()
export class Update{{EntityName}}UseCase {
  private readonly repository = inject({{EntityName}}RepositoryToken);
  // private readonly context = inject(APP_CONTEXT);

  async execute(request: Update{{EntityName}}Dto): Promise<Result<{{EntityName}}>> {
    const entityId = {{EntityName}}Id(request.Id);

    // 1. Fetch existing entity
    const result = await this.repository.findById(entityId);
    if (!result.isSuccess) {
      return Result.failure<{{EntityName}}>(
        result.errorMessage ?? 'Failed to retrieve {{entityName}}'
      );
    }

    const currentEntity = result.value!;

    // 2. Optional: Check for duplicate names (unique name validation)
    // Remove this block if your entity does not enforce unique names.
    const allEntitiesResult = await this.repository.findAll();
    if (allEntitiesResult.isSuccess) {
      const normalizedName = request.Name.trim().toLowerCase();
      const exists = allEntitiesResult.value!.some(
        e => e.id !== entityId &&
             e.name.toLowerCase() === normalizedName &&
             e.isActive
      );

      if (exists) {
        return Result.failure<{{EntityName}}>(
          `{{EntityName}} with name '${request.Name}' already exists.`
        );
      }
    }

    // 3. Validate updated entity using factory (re-runs all domain invariants)
    const updatedResult = create{{EntityName}}({
      ...currentEntity,
      name: request.Name,
      description: request.Description,
      position: request.Position ?? currentEntity.position,
      isActive: request.IsActive ?? currentEntity.isActive,
      // modifiedBy: UserId(this.context.userId),
      modifiedDate: new Date(),
    });

    if (!updatedResult.isSuccess) {
      return Result.failure<{{EntityName}}>(
        updatedResult.errorMessage ?? 'Unknown validation error'
      );
    }

    // 4. Persist the changes
    return this.repository.update(entityId, {
      name: request.Name,
      description: request.Description,
      position: request.Position ?? currentEntity.position,
      isActive: request.IsActive ?? currentEntity.isActive,
      // modifiedBy: UserId(this.context.userId),
      modifiedDate: new Date(),
    });
  }
}

// ============================================
// Delete Use Case
// ============================================

/**
 * Command use case to delete a {{EntityName}}.
 *
 * @example
 * ```typescript
 * const useCase = inject(Delete{{EntityName}}UseCase);
 * const result = await useCase.execute({{EntityName}}Id(1));
 * ```
 */
@Injectable()
export class Delete{{EntityName}}UseCase {
  private readonly repository = inject({{EntityName}}RepositoryToken);

  async execute(id: {{EntityName}}Id): Promise<Result<void>> {
    // Repository.delete() returns Result<void>
    return this.repository.delete(id);
  }
}
```

---

## Barrel Export

Create `<module-path>/src/lib/application/usecases/{{entityName}}/index.ts`:

```typescript
export * from './create-{{entityName}}.usecase';
export * from './update-{{entityName}}.usecase';
export * from './delete-{{entityName}}.usecase';
```

Update `<module-path>/src/lib/application/usecases/index.ts`:

```typescript
export * from './{{entityName}}';
```

---

## Registering Use Cases as Providers

Add use cases to the providers in the Infrastructure layer's providers file:

```typescript
export const {{entityName}}Providers: Provider[] = [
  // Repository token binding
  { provide: {{EntityName}}RepositoryToken, useClass: {{EntityName}}RepositoryImpl },
  // Command use cases
  Create{{EntityName}}UseCase,
  Update{{EntityName}}UseCase,
  Delete{{EntityName}}UseCase,
];
```

---

## Key Principles

1. **`@Injectable()`** — Required for Angular DI
2. **`inject()`** — Function-based DI, never constructor injection
3. **Repository via `InjectionToken`** — Never import the concrete implementation
4. **`Result<T>`** — Always return Result; never throw from a use case
5. **Domain factory for validation** — Use `create{{EntityName}}()` in create and update, not manual assignment
6. **File suffix** — `.usecase.ts`
7. **No context dependency on Infrastructure** — Use cases only depend on Domain interfaces

## Naming Conventions

| Type   | File Name                          | Class Name                  |
|--------|------------------------------------|-----------------------------|
| Create | `create-{{entityName}}.usecase.ts` | `Create{{EntityName}}UseCase` |
| Update | `update-{{entityName}}.usecase.ts` | `Update{{EntityName}}UseCase` |
| Delete | `delete-{{entityName}}.usecase.ts` | `Delete{{EntityName}}UseCase` |
