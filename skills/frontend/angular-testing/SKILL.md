---
name: angular-testing
description: Angular testing patterns with Jest and TestBed. Covers unit tests for use cases, queries, services, and components using AAA pattern, mock repositories via InjectionToken, and signal-based component testing. Use when creating .spec.ts files, writing tests, or when the user mentions testing. ALWAYS use when generating test files.
---

# Angular Testing Patterns

Jest + TestBed with AAA (Arrange/Act/Assert) pattern.

## Test File Location

Test files live next to the source file: `entity.service.ts` → `entity.service.spec.ts`

## Core Pattern: TestBed + InjectionToken Mocks

```typescript
import { TestBed } from '@angular/core/testing';

describe('CreateEntityUseCase', () => {
  let useCase: CreateEntityUseCase;
  let mockRepository: jest.Mocked<EntityRepository>;

  beforeEach(() => {
    mockRepository = {
      find: jest.fn(),
      findById: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };

    TestBed.configureTestingModule({
      providers: [
        CreateEntityUseCase,
        { provide: ENTITY_REPOSITORY, useValue: mockRepository },
        { provide: AuthService, useValue: { getCurrentUser: () => ({ id: 1 }) } },
      ],
    });

    useCase = TestBed.inject(CreateEntityUseCase);
  });

  it('should create entity successfully', async () => {
    // Arrange
    const request = { name: 'Test Entity' };
    const expected = { id: 1, name: 'Test Entity' };
    mockRepository.create.mockResolvedValue(Result.success(expected));

    // Act
    const result = await useCase.execute(request);

    // Assert
    expect(result.isSuccess).toBe(true);
    expect(result.value).toEqual(expected);
    expect(mockRepository.create).toHaveBeenCalled();
  });

  it('should return failure when name is empty', async () => {
    const result = await useCase.execute({ name: '' });
    expect(result.isFailure).toBe(true);
  });
});
```

## Signal Testing

```typescript
it('should update signal state', () => {
  const service = TestBed.inject(EntityListStateService);

  service.setKeyword('test');

  expect(service.keyword()).toBe('test');
  expect(service.page()).toBe(1); // reset on filter change
});
```

## Detailed Patterns

- UseCase/Query tests: [references/usecase-testing.md](references/usecase-testing.md)
- Component tests: [references/component-testing.md](references/component-testing.md)
- State service tests: [references/state-service-testing.md](references/state-service-testing.md)
