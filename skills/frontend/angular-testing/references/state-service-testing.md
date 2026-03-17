# State Service Testing

## Basic State Service Test

```typescript
describe('EntityListStateService', () => {
  let service: EntityListStateService;
  let mockQuery: jest.Mocked<GetEntitiesQuery>;

  beforeEach(() => {
    mockQuery = { execute: jest.fn() } as any;
    mockQuery.execute.mockResolvedValue(
      Result.success({ data: [], total: 0, page: 1, pageSize: 20 })
    );

    TestBed.configureTestingModule({
      providers: [
        EntityListStateService,
        { provide: GetEntitiesQuery, useValue: mockQuery },
        { provide: DeleteEntityUseCase, useValue: { execute: jest.fn() } },
        { provide: NotificationService, useValue: { success: jest.fn(), error: jest.fn() } },
      ],
    });

    service = TestBed.inject(EntityListStateService);
  });

  it('should initialize with defaults', () => {
    expect(service.keyword()).toBe('');
    expect(service.page()).toBe(1);
    expect(service.pageSize()).toBe(20);
  });

  it('should reset page when keyword changes', () => {
    service.setPage(3);
    service.setKeyword('test');
    expect(service.keyword()).toBe('test');
    expect(service.page()).toBe(1);
  });

  it('should reset page when page size changes', () => {
    service.setPage(5);
    service.setPageSize(50);
    expect(service.pageSize()).toBe(50);
    expect(service.page()).toBe(1);
  });

  it('should compute hasItems', async () => {
    mockQuery.execute.mockResolvedValue(
      Result.success({ data: [{ id: 1, name: 'A' }], total: 1 })
    );
    service.refresh();
    await new Promise(resolve => setTimeout(resolve, 200));
    expect(service.hasItems()).toBe(true);
  });
});
```

## Testing Delete with Notification

```typescript
it('should show success notification on delete', async () => {
  const mockDelete = TestBed.inject(DeleteEntityUseCase) as jest.Mocked<DeleteEntityUseCase>;
  const mockNotification = TestBed.inject(NotificationService) as jest.Mocked<NotificationService>;

  mockDelete.execute.mockResolvedValue(Result.success(undefined, 'Deleted'));
  mockQuery.execute.mockResolvedValue(Result.success({ data: [], total: 0 }));

  await service.deleteItem({ id: 1, name: 'Test' } as Entity);

  expect(mockNotification.success).toHaveBeenCalledWith('Deleted');
});
```
