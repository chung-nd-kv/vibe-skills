# Angular TDD Patterns

Concrete test patterns for Angular applications using Jest + TestBed with AAA (Arrange-Act-Assert).

---

## Component Testing

### Smart Component (with dependencies)

```typescript
import { TestBed } from '@angular/core/testing';
import { EntityListComponent } from './entity-list.component';
import { ENTITY_REPOSITORY } from '../../domain/repositories/entity.repository';
import { GetEntitiesQuery } from '../../application/queries/get-entities/get-entities.query';

describe('EntityListComponent', () => {
  let component: EntityListComponent;
  let fixture: ComponentFixture<EntityListComponent>;
  let mockQuery: jest.Mocked<GetEntitiesQuery>;

  beforeEach(() => {
    mockQuery = {
      execute: jest.fn(),
    } as any;

    TestBed.configureTestingModule({
      imports: [EntityListComponent],
      providers: [
        { provide: GetEntitiesQuery, useValue: mockQuery },
      ],
    });

    fixture = TestBed.createComponent(EntityListComponent);
    component = fixture.componentInstance;
  });

  it('should load entities on init', async () => {
    // Arrange
    const entities = [{ id: 1, name: 'Entity 1' }];
    mockQuery.execute.mockResolvedValue(Result.success({ items: entities, total: 1 }));

    // Act
    fixture.detectChanges(); // triggers ngOnInit / constructor effects

    // Assert
    expect(mockQuery.execute).toHaveBeenCalled();
  });

  it('should display loading state while fetching', () => {
    // Arrange
    mockQuery.execute.mockReturnValue(new Promise(() => {})); // never resolves

    // Act
    fixture.detectChanges();

    // Assert
    const loading = fixture.nativeElement.querySelector('[data-testid="loading"]');
    expect(loading).toBeTruthy();
  });

  it('should display empty state when no entities', async () => {
    // Arrange
    mockQuery.execute.mockResolvedValue(Result.success({ items: [], total: 0 }));

    // Act
    fixture.detectChanges();
    await fixture.whenStable();
    fixture.detectChanges();

    // Assert
    const empty = fixture.nativeElement.querySelector('[data-testid="empty-state"]');
    expect(empty).toBeTruthy();
  });
});
```

### Dumb Component (inputs/outputs)

```typescript
describe('EntityCardComponent', () => {
  let component: EntityCardComponent;
  let fixture: ComponentFixture<EntityCardComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [EntityCardComponent],
    });
    fixture = TestBed.createComponent(EntityCardComponent);
    component = fixture.componentInstance;
  });

  it('should display entity name', () => {
    // Arrange
    fixture.componentRef.setInput('entity', { id: 1, name: 'Test Entity' });

    // Act
    fixture.detectChanges();

    // Assert
    const name = fixture.nativeElement.querySelector('[data-testid="entity-name"]');
    expect(name.textContent).toContain('Test Entity');
  });

  it('should emit delete event when delete button clicked', () => {
    // Arrange
    fixture.componentRef.setInput('entity', { id: 1, name: 'Test' });
    fixture.detectChanges();
    const deleteSpy = jest.fn();
    component.deleted.subscribe(deleteSpy);

    // Act
    const button = fixture.nativeElement.querySelector('[data-testid="delete-btn"]');
    button.click();

    // Assert
    expect(deleteSpy).toHaveBeenCalledWith(1);
  });
});
```

### Signal Input Testing

```typescript
it('should react to signal input changes', () => {
  // Arrange
  fixture.componentRef.setInput('title', 'Initial');
  fixture.detectChanges();

  // Act
  fixture.componentRef.setInput('title', 'Updated');
  fixture.detectChanges();

  // Assert
  const heading = fixture.nativeElement.querySelector('h1');
  expect(heading.textContent).toContain('Updated');
});
```

---

## UseCase / Query Testing

```typescript
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
      ],
    });

    useCase = TestBed.inject(CreateEntityUseCase);
  });

  it('should create entity with valid data', async () => {
    // Arrange
    const request = { name: 'New Entity' };
    const expected = { id: 1, name: 'New Entity' };
    mockRepository.create.mockResolvedValue(Result.success(expected));

    // Act
    const result = await useCase.execute(request);

    // Assert
    expect(result.isSuccess).toBe(true);
    expect(result.value).toEqual(expected);
    expect(mockRepository.create).toHaveBeenCalledWith(
      expect.objectContaining({ name: 'New Entity' })
    );
  });

  it('should return failure when name is empty', async () => {
    // Act
    const result = await useCase.execute({ name: '' });

    // Assert
    expect(result.isFailure).toBe(true);
    expect(result.errorMessage).toContain('Name');
  });

  it('should return failure when repository fails', async () => {
    // Arrange
    mockRepository.create.mockResolvedValue(Result.failure('Database error'));

    // Act
    const result = await useCase.execute({ name: 'Valid' });

    // Assert
    expect(result.isFailure).toBe(true);
  });
});
```

---

## State Service Testing

```typescript
describe('EntityListStateService', () => {
  let service: EntityListStateService;
  let mockQuery: jest.Mocked<GetEntitiesQuery>;

  beforeEach(() => {
    mockQuery = { execute: jest.fn() } as any;

    TestBed.configureTestingModule({
      providers: [
        EntityListStateService,
        { provide: GetEntitiesQuery, useValue: mockQuery },
      ],
    });

    service = TestBed.inject(EntityListStateService);
  });

  it('should initialize with default state', () => {
    expect(service.entities()).toEqual([]);
    expect(service.loading()).toBe(false);
    expect(service.page()).toBe(1);
    expect(service.keyword()).toBe('');
  });

  it('should update keyword and reset page', () => {
    // Arrange
    service.setPage(3);

    // Act
    service.setKeyword('test');

    // Assert
    expect(service.keyword()).toBe('test');
    expect(service.page()).toBe(1); // reset on filter change
  });

  it('should set loading state during fetch', async () => {
    // Arrange
    mockQuery.execute.mockResolvedValue(Result.success({ items: [], total: 0 }));

    // Act
    const promise = service.loadEntities();

    // Assert — loading is true during fetch
    expect(service.loading()).toBe(true);

    await promise;

    // Assert — loading is false after fetch
    expect(service.loading()).toBe(false);
  });

  it('should update entities after successful fetch', async () => {
    // Arrange
    const entities = [{ id: 1, name: 'Entity 1' }];
    mockQuery.execute.mockResolvedValue(Result.success({ items: entities, total: 1 }));

    // Act
    await service.loadEntities();

    // Assert
    expect(service.entities()).toEqual(entities);
    expect(service.total()).toBe(1);
  });
});
```

---

## Form Testing

```typescript
describe('EntityFormComponent', () => {
  let component: EntityFormComponent;
  let fixture: ComponentFixture<EntityFormComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [EntityFormComponent, ReactiveFormsModule],
    });
    fixture = TestBed.createComponent(EntityFormComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should initialize with empty form', () => {
    expect(component.form.value).toEqual({ name: '', description: '' });
    expect(component.form.valid).toBe(false);
  });

  it('should validate required name field', () => {
    // Act
    component.form.controls.name.setValue('');
    component.form.controls.name.markAsTouched();

    // Assert
    expect(component.form.controls.name.hasError('required')).toBe(true);
  });

  it('should validate name max length', () => {
    // Act
    component.form.controls.name.setValue('a'.repeat(101));

    // Assert
    expect(component.form.controls.name.hasError('maxlength')).toBe(true);
  });

  it('should emit form value on valid submission', () => {
    // Arrange
    const submitSpy = jest.fn();
    component.submitted.subscribe(submitSpy);
    component.form.setValue({ name: 'Test', description: 'Desc' });

    // Act
    component.onSubmit();

    // Assert
    expect(submitSpy).toHaveBeenCalledWith({ name: 'Test', description: 'Desc' });
  });

  it('should not emit on invalid submission', () => {
    // Arrange
    const submitSpy = jest.fn();
    component.submitted.subscribe(submitSpy);

    // Act
    component.onSubmit();

    // Assert
    expect(submitSpy).not.toHaveBeenCalled();
  });

  it('should display validation errors after submit attempt', () => {
    // Act
    component.onSubmit();
    fixture.detectChanges();

    // Assert
    const error = fixture.nativeElement.querySelector('[data-testid="name-error"]');
    expect(error).toBeTruthy();
  });
});
```

---

## HTTP Testing

```typescript
import { provideHttpClient } from '@angular/common/http';
import { HttpTestingController, provideHttpClientTesting } from '@angular/common/http/testing';

describe('EntityApiClient', () => {
  let apiClient: EntityApiClient;
  let httpController: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        EntityApiClient,
        provideHttpClient(),
        provideHttpClientTesting(),
      ],
    });

    apiClient = TestBed.inject(EntityApiClient);
    httpController = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpController.verify(); // ensure no outstanding requests
  });

  it('should fetch entities with correct URL and params', () => {
    // Act
    apiClient.getEntities({ page: 1, keyword: 'test' }).subscribe();

    // Assert
    const req = httpController.expectOne(
      r => r.url === '/api/entities' && r.params.get('page') === '1'
    );
    expect(req.request.method).toBe('GET');
    expect(req.request.params.get('keyword')).toBe('test');

    req.flush({ items: [], total: 0 });
  });

  it('should map response to domain model', (done) => {
    // Act
    apiClient.getEntityById(1).subscribe(result => {
      // Assert
      expect(result.isSuccess).toBe(true);
      expect(result.value).toEqual({ id: 1, name: 'Test' });
      done();
    });

    httpController.expectOne('/api/entities/1').flush({ id: 1, name: 'Test' });
  });

  it('should handle HTTP error', (done) => {
    // Act
    apiClient.getEntityById(999).subscribe(result => {
      // Assert
      expect(result.isFailure).toBe(true);
      done();
    });

    httpController.expectOne('/api/entities/999').flush(
      { message: 'Not found' },
      { status: 404, statusText: 'Not Found' }
    );
  });
});
```

---

## Pipe Testing

```typescript
describe('CurrencyFormatPipe', () => {
  let pipe: CurrencyFormatPipe;

  beforeEach(() => {
    pipe = new CurrencyFormatPipe();
  });

  it('should format number with currency symbol', () => {
    expect(pipe.transform(1000, 'VND')).toBe('1,000 ₫');
  });

  it('should handle null value', () => {
    expect(pipe.transform(null, 'VND')).toBe('—');
  });

  it('should handle zero', () => {
    expect(pipe.transform(0, 'VND')).toBe('0 ₫');
  });
});
```

---

## Directive Testing

```typescript
describe('HighlightDirective', () => {
  @Component({
    standalone: true,
    imports: [HighlightDirective],
    template: `<span appHighlight [keyword]="keyword">Hello World</span>`,
  })
  class TestHostComponent {
    keyword = '';
  }

  let fixture: ComponentFixture<TestHostComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [TestHostComponent],
    });
    fixture = TestBed.createComponent(TestHostComponent);
  });

  it('should highlight matching text', () => {
    // Arrange
    fixture.componentInstance.keyword = 'World';

    // Act
    fixture.detectChanges();

    // Assert
    const highlighted = fixture.nativeElement.querySelector('mark');
    expect(highlighted.textContent).toBe('World');
  });

  it('should not highlight when keyword is empty', () => {
    // Act
    fixture.detectChanges();

    // Assert
    const highlighted = fixture.nativeElement.querySelector('mark');
    expect(highlighted).toBeNull();
  });
});
```

---

## Common Test Utilities

### Test Data Factory

```typescript
// test-utils/entity.factory.ts
export function createTestEntity(overrides: Partial<Entity> = {}): Entity {
  return {
    id: EntityId(1),
    name: 'Test Entity',
    isActive: true,
    ...overrides,
  };
}

export function createTestEntityList(count: number): Entity[] {
  return Array.from({ length: count }, (_, i) =>
    createTestEntity({ id: EntityId(i + 1), name: `Entity ${i + 1}` })
  );
}
```

### Custom Component Creator

```typescript
// test-utils/component.helper.ts
export function createComponent<T>(
  component: Type<T>,
  providers: Provider[] = []
): { fixture: ComponentFixture<T>; component: T } {
  TestBed.configureTestingModule({
    imports: [component],
    providers,
  });

  const fixture = TestBed.createComponent(component);
  return { fixture, component: fixture.componentInstance };
}
```

### Async Helpers

```typescript
// Using fakeAsync for timer-based tests
it('should debounce search input', fakeAsync(() => {
  // Arrange
  const searchSpy = jest.fn();
  component.search.subscribe(searchSpy);

  // Act
  component.onSearchInput('te');
  tick(200); // not enough time
  component.onSearchInput('test');
  tick(300); // debounce completes

  // Assert
  expect(searchSpy).toHaveBeenCalledTimes(1);
  expect(searchSpy).toHaveBeenCalledWith('test');
}));

// Using fixture.whenStable() for async operations
it('should load data after init', async () => {
  fixture.detectChanges();
  await fixture.whenStable();
  fixture.detectChanges();

  expect(component.entities().length).toBeGreaterThan(0);
});
```
