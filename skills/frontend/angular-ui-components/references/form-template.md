# Form Component Template

Full annotated template for a presentation layer form component supporting both create and edit modes via signal inputs.

## Variable Substitution

| Placeholder | Format | Example |
|---|---|---|
| `{{EntityName}}` | PascalCase | `Product` |
| `{{entityName}}` | camelCase | `product` |
| `{{entity-name}}` | kebab-case | `product` |
| `{{ModuleName}}` | PascalCase | `Inventory` |
| `{{module-name}}` | kebab-case | `inventory` |

## TypeScript — `{{entity-name}}-form.component.ts`

```typescript
import {
  Component,
  ChangeDetectionStrategy,
  OnInit,
  AfterViewInit,
  inject,
  input,
  output,
  computed,
  viewChild,
  ElementRef,
  signal,
} from '@angular/core';
import { ReactiveFormsModule, FormGroup, FormControl, Validators } from '@angular/forms';
import { toSignal } from '@angular/core/rxjs-interop';

// Import your UI library's button and form control components, for example:
// import { MatButtonModule } from '@angular/material/button';
// import { MatInputModule } from '@angular/material/input';

// Import your toast/notification service if applicable:
// import { ToastService } from '@your-org/notifications';

// Import your modal control token/service if this component is used inside a modal:
// import { ModalControl } from '@your-org/modal';
// The ModalControl pattern allows the form to close the parent modal with a result.

// Import translation pipe if your project uses i18n:
// import { TranslatePipe } from '@your-org/localization';

import { {{ModuleName}}StateService } from '../../services/{{module-name}}.state.service';
import { {{EntityName}} } from '../../../domain/entities/{{entity-name}}.entity';

// ─── Form Control Interface ──────────────────────────────────────────────────
// Always type your FormGroup — it provides compile-time safety for form values.
// Add one entry per form field.
interface {{EntityName}}FormControls {
  name: FormControl<string>;
  description: FormControl<string>;
  // Add more typed controls here as needed:
  // isActive: FormControl<boolean>;
  // categoryId: FormControl<string | null>;
}

@Component({
  selector: 'app-{{entity-name}}-form',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    ReactiveFormsModule,
    // Add your UI library's components here:
    // MatButtonModule, MatInputModule, TranslatePipe, etc.
  ],
  templateUrl: './{{entity-name}}-form.component.html',
  styleUrl: './{{entity-name}}-form.component.scss',
})
export class {{EntityName}}FormComponent implements OnInit, AfterViewInit {
  // ─── Inputs ───────────────────────────────────────────────────────────────
  // mode is required — the parent must always specify 'create' or 'edit'
  readonly mode = input.required<'create' | 'edit'>();

  // entity is optional — used in edit mode to pre-populate the form
  readonly entity = input<{{EntityName}} | null>(null);

  // entityId is optional — used when only the ID is available (lazy load scenario)
  readonly entityId = input<string | null>(null);

  // ─── Outputs ──────────────────────────────────────────────────────────────
  // Emit the saved entity when submission succeeds
  readonly saved = output<{{EntityName}}>();

  // Emit void when the user cancels without saving
  readonly cancelled = output<void>();

  // ─── Dependencies ─────────────────────────────────────────────────────────
  private readonly state = inject({{ModuleName}}StateService);

  // Inject your modal control if this form is used inside a modal dialog:
  // private readonly modalControl = inject(ModalControl, { optional: true });

  // Inject toast service for user feedback:
  // private readonly toast = inject(ToastService);

  // ─── Focus Management ─────────────────────────────────────────────────────
  // viewChild gets a reference to the first form field for auto-focus
  readonly firstField = viewChild<ElementRef<HTMLInputElement>>('firstField');

  // ─── Reactive Form ────────────────────────────────────────────────────────
  readonly form = new FormGroup<{{EntityName}}FormControls>({
    name: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required, Validators.maxLength(100)],
    }),
    description: new FormControl('', {
      nonNullable: true,
      validators: [Validators.maxLength(500)],
    }),
    // Add more controls here:
    // isActive: new FormControl(true, { nonNullable: true }),
  });

  // ─── Reactive Form Status as Signal ───────────────────────────────────────
  // toSignal() converts the statusChanges observable to a signal,
  // enabling use in computed() without subscribe/unsubscribe boilerplate.
  private readonly formStatus = toSignal(this.form.statusChanges, {
    initialValue: this.form.status,
  });

  // ─── Local UI State ───────────────────────────────────────────────────────
  private readonly submitting = signal(false);

  // ─── Computed Signals ─────────────────────────────────────────────────────
  // canSubmit is derived from form validity + submitting state.
  // Use this in the template to disable the submit button.
  readonly canSubmit = computed(() =>
    this.formStatus() === 'VALID' && !this.submitting()
  );

  readonly isEditMode = computed(() => this.mode() === 'edit');

  readonly submitLabel = computed(() =>
    this.isEditMode() ? 'Save Changes' : 'Create'
    // Or use translation keys:
    // this.isEditMode() ? 'form.button.save' : 'form.button.create'
  );

  // ─── Lifecycle ────────────────────────────────────────────────────────────
  ngOnInit(): void {
    // Pre-populate the form when in edit mode
    const existing = this.entity();
    if (existing && this.mode() === 'edit') {
      this.form.patchValue({
        name: existing.name,
        description: (existing as any).description ?? '',
        // Map all entity fields to their form controls
      });
    }
  }

  ngAfterViewInit(): void {
    // Auto-focus the first field after the view is initialized.
    // setTimeout pushes focus to the next microtask to avoid ExpressionChangedAfterItHasBeenChecked.
    setTimeout(() => {
      this.firstField()?.nativeElement.focus();
    }, 50);
  }

  // ─── Event Handlers ───────────────────────────────────────────────────────

  onSubmit(): void {
    // Mark all fields as touched to trigger validation display
    this.form.markAllAsTouched();
    if (!this.canSubmit()) return;

    if (this.mode() === 'create') {
      this.handleCreate();
    } else {
      this.handleUpdate();
    }
  }

  private async handleCreate(): Promise<void> {
    this.submitting.set(true);
    try {
      const formValue = this.form.getRawValue();
      const result = await this.state.create(formValue);
      if (result) {
        // Option A: Emit to parent
        this.saved.emit(result);

        // Option B: Close modal with result (if using modal control pattern)
        // this.modalControl?.close(result);

        // Show success toast if applicable:
        // this.toast.success('Item created successfully');
      }
    } catch (err) {
      console.error('[{{EntityName}}FormComponent] Create error:', err);
      // this.toast.error('Failed to create item');
    } finally {
      this.submitting.set(false);
    }
  }

  private async handleUpdate(): Promise<void> {
    // Resolve the ID from either entityId input or the entity object
    const id = this.entityId() ?? this.entity()?.id;
    if (!id) {
      console.error('[{{EntityName}}FormComponent] Cannot update: no entity ID');
      return;
    }

    this.submitting.set(true);
    try {
      const formValue = this.form.getRawValue();
      const result = await this.state.update(id, formValue);
      if (result) {
        // Option A: Emit to parent
        this.saved.emit(result);

        // Option B: Close modal with result (if using modal control pattern)
        // this.modalControl?.close(result);

        // Show success toast if applicable:
        // this.toast.success('Item updated successfully');
      }
    } catch (err) {
      console.error('[{{EntityName}}FormComponent] Update error:', err);
      // this.toast.error('Failed to update item');
    } finally {
      this.submitting.set(false);
    }
  }

  onCancel(): void {
    this.cancelled.emit();

    // Close modal without result (if using modal control pattern):
    // this.modalControl?.close(null);
  }
}
```

## HTML Template — `{{entity-name}}-form.component.html`

```html
<!-- Form container -->
<div class="form-container">

  <!-- Form header -->
  <div class="form-header">
    <h2>
      @if (isEditMode()) {
        Edit <!-- EntityName -->
      } @else {
        Create <!-- EntityName -->
      }
    </h2>
  </div>

  <!-- Reactive Form -->
  <form [formGroup]="form" (ngSubmit)="onSubmit()" novalidate>

    <!-- Name field -->
    <!-- Replace the input below with your library's form control component -->
    <div class="form-field">
      <label for="name">Name *</label>
      <input
        #firstField
        id="name"
        type="text"
        formControlName="name"
        placeholder="Enter name"
        [attr.aria-invalid]="form.controls.name.invalid && form.controls.name.touched"
      />
      @if (form.controls.name.hasError('required') && form.controls.name.touched) {
        <span class="field-error">Name is required</span>
      }
      @if (form.controls.name.hasError('maxlength') && form.controls.name.touched) {
        <span class="field-error">Name must be 100 characters or fewer</span>
      }
    </div>

    <!-- Description field -->
    <div class="form-field">
      <label for="description">Description</label>
      <textarea
        id="description"
        formControlName="description"
        placeholder="Enter description (optional)"
        rows="3"
      ></textarea>
      @if (form.controls.description.hasError('maxlength') && form.controls.description.touched) {
        <span class="field-error">Description must be 500 characters or fewer</span>
      }
    </div>

    <!-- Add more fields here using the same pattern -->

    <!-- Form Actions -->
    <div class="form-actions">
      <!-- Cancel button -->
      <!-- Replace with your library's button component -->
      <button type="button" (click)="onCancel()">
        Cancel
      </button>

      <!-- Submit button — disabled when canSubmit() is false -->
      <!-- Replace with your library's button component -->
      <button
        type="submit"
        [disabled]="!canSubmit()"
      >
        {{ submitLabel() }}
      </button>
    </div>

  </form>

</div>
```

## Modal Control Pattern

If your project uses a modal service that injects a `ModalControl` token into modal content components, wire it up as follows:

```typescript
// In the form component constructor or field:
// private readonly modalControl = inject(ModalControl, { optional: true });

// In handleCreate / handleUpdate, instead of emitting:
// this.modalControl?.close(result);

// In onCancel:
// this.modalControl?.close(null);
```

The calling code (page component or modal opener) then receives the result:

```typescript
// In the page component:
const result = await this.modalService.open({{EntityName}}FormComponent, {
  mode: 'create',
});
if (result) {
  this.state.load();
}
```

## Checklist

- [ ] `standalone: true`
- [ ] `ChangeDetectionStrategy.OnPush`
- [ ] `mode = input.required<'create' | 'edit'>()` — never a plain string prop
- [ ] `entity = input<{{EntityName}} | null>(null)` for pre-population
- [ ] Typed `FormGroup<{{EntityName}}FormControls>` — not untyped `FormGroup`
- [ ] `toSignal(form.statusChanges)` with `initialValue` — not `.subscribe()`
- [ ] `computed canSubmit` — combines `formStatus() === 'VALID'` and `!submitting()`
- [ ] `viewChild` for first-field auto-focus
- [ ] `form.markAllAsTouched()` before checking validity in `onSubmit`
- [ ] `handleCreate` and `handleUpdate` are private methods — `onSubmit` delegates
- [ ] `submitting` is a `signal()` — not a plain boolean
- [ ] Component exported from `presentation/components/index.ts`
