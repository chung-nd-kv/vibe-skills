# UseCase & Query Testing

## Query Test

```typescript
describe('GetEntitiesQuery', () => {
  let query: GetEntitiesQuery;
  let mockRepo: jest.Mocked<EntityRepository>;

  beforeEach(() => {
    mockRepo = { find: jest.fn(), create: jest.fn(), update: jest.fn(), delete: jest.fn() };
    TestBed.configureTestingModule({
      providers: [GetEntitiesQuery, { provide: ENTITY_REPOSITORY, useValue: mockRepo }],
    });
    query = TestBed.inject(GetEntitiesQuery);
  });

  it('should return paged results', async () => {
    const expected = { data: [{ id: 1, name: 'A' }], total: 1 };
    mockRepo.find.mockResolvedValue(Result.success(expected));

    const result = await query.execute({ page: 1, pageSize: 20 });

    expect(result.isSuccess).toBe(true);
    expect(result.value?.data).toHaveLength(1);
  });
});
```

## Create UseCase Test

```typescript
describe('CreateEntityUseCase', () => {
  let useCase: CreateEntityUseCase;
  let mockRepo: jest.Mocked<EntityRepository>;

  beforeEach(() => {
    mockRepo = { find: jest.fn(), create: jest.fn(), update: jest.fn(), delete: jest.fn() };
    TestBed.configureTestingModule({
      providers: [
        CreateEntityUseCase,
        { provide: ENTITY_REPOSITORY, useValue: mockRepo },
        { provide: AuthService, useValue: { getCurrentUser: () => ({ id: 1, tenantId: 1 }) } },
      ],
    });
    useCase = TestBed.inject(CreateEntityUseCase);
  });

  it('should create and return entity', async () => {
    mockRepo.create.mockResolvedValue(Result.success({ id: 1, name: 'New' }));
    const result = await useCase.execute({ name: 'New' });
    expect(result.isSuccess).toBe(true);
    expect(result.value?.name).toBe('New');
  });

  it('should fail on validation error', async () => {
    const result = await useCase.execute({ name: '' });
    expect(result.isFailure).toBe(true);
  });

  it('should fail on repository error', async () => {
    mockRepo.create.mockResolvedValue(Result.failure('DB error'));
    const result = await useCase.execute({ name: 'Valid' });
    expect(result.isFailure).toBe(true);
  });
});
```
