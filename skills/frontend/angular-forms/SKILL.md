---
name: angular-forms
description: Angular reactive forms with typed FormGroup/FormControl, signal-based validation, dialog form integration, and exhaustMap submission. Use when creating forms, dialog forms, validation, or form submission. ALWAYS use when implementing dialog form patterns, reactive form patterns, or form submission with exhaustMap.
---

# Angular Reactive Forms

Typed reactive forms with signal-based status tracking, commonly used inside dialog components.

## Core Form Setup

```typescript
import { FormControl, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { toSignal } from '@angular/core/rxjs-interop';
import { startWith } from 'rxjs';

readonly form = new FormGroup({
  name: new FormControl('', {
    nonNullable: true,
    validators: [Validators.required, Validators.maxLength(255)],
  }),
  description: new FormControl('', { nonNullable: true }),
  isActive: new FormControl(true, { nonNullable: true }),
});
```

## Signal-Based Form Status

```typescript
// MUST use startWith to capture initial status
private readonly formStatus = toSignal(
  this.form.statusChanges.pipe(startWith(this.form.status)),
  { initialValue: this.form.status }
);

readonly saveDisabled = computed(() =>
  this.formStatus() !== 'VALID' || this._submitting()
);
```

## Computed Error Messages

```typescript
readonly nameErrorMessage = computed(() => {
  const control = this.form.controls.name;
  if (!control.touched) return undefined;
  if (control.hasError('required')) return 'Name is required';
  if (control.hasError('maxlength')) return 'Name must be 255 characters or less';
  return undefined;
});
```

## Submission with exhaustMap

Prevent double-submit using Subject + exhaustMap:

```typescript
private readonly submitTrigger$ = new Subject<void>();
private readonly destroyRef = inject(DestroyRef);

ngOnInit(): void {
  this.submitTrigger$
    .pipe(
      exhaustMap(() => this.handleSubmission()),
      takeUntilDestroyed(this.destroyRef)  // MUST pass destroyRef
    )
    .subscribe();
}

onSubmit(): void {
  if (this.form.invalid) { this.markFormAsTouched(); return; }
  if (this._submitting()) return;
  this.submitTrigger$.next();
}

private markFormAsTouched(): void {
  Object.values(this.form.controls).forEach(c => c.markAsTouched());
}
```

## Dual Mode (Create/Edit)

```typescript
mode = input.required<'create' | 'edit'>();
entity = input<Entity>();

readonly isCreateMode = computed(() => this.mode() === 'create');

ngOnInit(): void {
  // ... wire submitTrigger$ ...

  if (this.mode() === 'edit') {
    const existing = this.entity();
    if (existing) {
      this.form.patchValue({ name: existing.name, description: existing.description });
    }
  }
}
```

## Key Rules

- `toSignal(form.statusChanges.pipe(startWith(form.status)))` — always use startWith
- Pass `this.destroyRef` to `takeUntilDestroyed()` in `ngOnInit()`
- Use `exhaustMap` for submission (prevents concurrent submits)
- Check `touched` before showing errors

For full dialog form patterns, see [references/form-patterns.md](references/form-patterns.md).
