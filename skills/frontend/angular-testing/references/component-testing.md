# Component Testing

## Page Component Test

```typescript
describe('EntityPageComponent', () => {
  let fixture: ComponentFixture<EntityPageComponent>;
  let component: EntityPageComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [EntityPageComponent],
      providers: [
        { provide: ENTITY_REPOSITORY, useValue: createMockRepository() },
        { provide: AuthService, useValue: { getCurrentUser: () => ({ id: 1 }) } },
        GetEntitiesQuery, CreateEntityUseCase, UpdateEntityUseCase, DeleteEntityUseCase,
        EntityApiClient,
      ],
    }).compileComponents();

    fixture = TestBed.createComponent(EntityPageComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

## Form Component Test

```typescript
describe('EntityFormComponent', () => {
  let fixture: ComponentFixture<EntityFormComponent>;
  let component: EntityFormComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [EntityFormComponent],
      providers: [
        CreateEntityUseCase, UpdateEntityUseCase,
        { provide: ENTITY_REPOSITORY, useValue: createMockRepository() },
        { provide: DialogControl, useValue: { close: jest.fn(), submit: jest.fn() } },
        { provide: NotificationService, useValue: { success: jest.fn(), error: jest.fn() } },
      ],
    }).compileComponents();

    fixture = TestBed.createComponent(EntityFormComponent);
    component = fixture.componentInstance;
    fixture.componentRef.setInput('mode', 'create');
    fixture.detectChanges();
  });

  it('should have invalid form initially', () => {
    expect(component.form.valid).toBe(false);
  });

  it('should become valid with name', () => {
    component.form.controls.name.setValue('Test');
    expect(component.form.valid).toBe(true);
  });
});
```

## Testing Signal Inputs

```typescript
it('should accept signal input', () => {
  fixture.componentRef.setInput('dataSource', [{ id: 1, name: 'A' }]);
  fixture.detectChanges();
  expect(component.dataSource()).toHaveLength(1);
});
```
