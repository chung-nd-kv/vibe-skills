# Form Patterns

## Dialog Form Component

Form components used inside dialogs typically implement a dialog content interface with template projection:

```typescript
@Component({
  selector: 'app-entity-form',
  imports: [CommonModule, ReactiveFormsModule],
  templateUrl: './entity-form.component.html',
  styleUrl: './entity-form.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class EntityFormComponent implements OnInit, AfterViewInit {
  // Template projection for dialog body/footer
  @ViewChild('bodyTemplate', { static: true, read: TemplateRef })
  bodyTemplate?: TemplateRef<unknown>;

  @ViewChild('footerTemplate', { static: true, read: TemplateRef })
  footerTemplate?: TemplateRef<unknown>;

  // Services
  private readonly dialogControl = inject(DialogControl, { optional: true });
  private readonly createUseCase = inject(CreateEntityUseCase);
  private readonly updateUseCase = inject(UpdateEntityUseCase);
  private readonly notification = inject(NotificationService);
  private readonly destroyRef = inject(DestroyRef);

  // Streams
  private readonly submitTrigger$ = new Subject<void>();
  private readonly nameInputRef = viewChild<ElementRef<HTMLInputElement>>('nameInput');

  // Inputs
  mode = input.required<'create' | 'edit'>();
  entity = input<Entity>();

  // Form
  readonly form = new FormGroup({
    name: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required, Validators.maxLength(255)],
    }),
  });

  // State
  private readonly _submitting = signal(false);
  readonly submitting = this._submitting.asReadonly();

  // Form status — MUST use startWith
  private readonly formStatus = toSignal(
    this.form.statusChanges.pipe(startWith(this.form.status)),
    { initialValue: this.form.status }
  );
  readonly saveDisabled = computed(() =>
    this.formStatus() !== 'VALID' || this._submitting()
  );

  // Error computeds
  readonly nameErrorMessage = computed(() => {
    const control = this.form.controls.name;
    if (!control.touched) return undefined;
    if (control.hasError('maxlength')) return 'Name must be 255 characters or less';
    return undefined;
  });

  ngOnInit(): void {
    this.submitTrigger$
      .pipe(
        exhaustMap(() => this.handleSubmission()),
        takeUntilDestroyed(this.destroyRef)
      )
      .subscribe();

    if (this.mode() === 'edit') {
      const existing = this.entity();
      if (existing) {
        this.form.patchValue({ name: existing.name });
      }
    }
  }

  ngAfterViewInit(): void {
    setTimeout(() => this.nameInputRef()?.nativeElement?.focus(), 100);
  }

  onCancel(): void { this.dialogControl?.close(); }
  onSubmit(): void {
    if (this.form.invalid) {
      Object.values(this.form.controls).forEach(c => c.markAsTouched());
      return;
    }
    if (this._submitting()) return;
    this.submitTrigger$.next();
  }

  private async handleSubmission(): Promise<void> {
    this._submitting.set(true);
    try {
      const result = this.mode() === 'create'
        ? await this.createUseCase.execute(this.form.getRawValue())
        : await this.updateUseCase.execute({ id: this.entity()!.id, ...this.form.getRawValue() });

      if (result.isSuccess) {
        this.notification.success(result.successMessage ?? 'Saved');
        this.dialogControl?.submit(result.value);
      } else {
        this.notification.error(result.errorMessage ?? 'Failed');
      }
    } finally {
      this._submitting.set(false);
    }
  }
}
```

## Form HTML Template

```html
<ng-template #bodyTemplate>
  <form [formGroup]="form" (keydown.enter)="onSubmit()">
    <div class="form-group">
      <label>Name <span class="required">*</span></label>
      <input #nameInput type="text" formControlName="name" maxlength="255" />
      @if (nameErrorMessage()) {
        <span class="error">{{ nameErrorMessage() }}</span>
      }
    </div>
  </form>
</ng-template>

<ng-template #footerTemplate>
  <div class="dialog-footer">
    <button type="button" (click)="onCancel()">Cancel</button>
    <button type="button" [disabled]="saveDisabled()" (click)="onSubmit()">
      @if (submitting()) { <span class="spinner"></span> }
      Save
    </button>
  </div>
</ng-template>
```

## Common Mistakes

1. Missing `startWith()` in form status signal → form appears invalid initially
2. Using `takeUntilDestroyed()` without `destroyRef` in `ngOnInit()` → throws error
3. Not checking `touched` before showing error → errors show immediately
4. Using `switchMap` instead of `exhaustMap` for submit → allows concurrent submits
