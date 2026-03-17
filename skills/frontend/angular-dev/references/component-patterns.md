# Component Patterns — Angular 17+

## Standalone Component anatomy

```typescript
@Component({
  selector: 'app-product-card',
  standalone: true,
  imports: [
    CommonModule,
    RouterModule,
    CurrencyPipe,
    // feature-specific imports
  ],
  templateUrl: './product-card.component.html',
  styleUrl: './product-card.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,  // always with Signals
})
export class ProductCardComponent {
  // ── Inputs ──────────────────────────────────────
  product = input.required<ProductDto>();           // Angular 17 input()
  showActions = input(true);                        // with default

  // ── Outputs ─────────────────────────────────────
  addToCart = output<ProductDto>();
  viewDetail = output<number>();

  // ── Injected services ────────────────────────────
  private router = inject(Router);

  // ── Derived state ────────────────────────────────
  isOnSale = computed(() => this.product().discountPercent > 0);

  // ── Actions ──────────────────────────────────────
  onAddToCart() {
    this.addToCart.emit(this.product());
  }
}
```

## Smart vs Dumb component split

```
ProductListPageComponent (Smart — container)
  ├── Injects store / service
  ├── Handles routing
  └── ProductCardComponent (Dumb — presentational)
        ├── Input: product
        └── Output: events only
```

**Rule:** Dumb components never inject stores or services.

---

## Template patterns

### Signals in template
```html
<!-- ✅ call signal as function in template -->
<p>{{ product().name }}</p>
<p>{{ total() | currency }}</p>

@if (isLoading()) {
  <app-spinner />
} @else {
  <app-product-list [items]="products()" />
}
```

### New control flow (@if, @for, @switch)
```html
<!-- @for with track -->
@for (item of items(); track item.id) {
  <app-product-card [product]="item" (addToCart)="onAdd($event)" />
} @empty {
  <p>No products found.</p>
}

<!-- @switch -->
@switch (status()) {
  @case ('loading') { <app-spinner /> }
  @case ('error')   { <app-error [message]="error()" /> }
  @default          { <app-product-list [items]="products()" /> }
}
```

### Async pipe vs toSignal
```typescript
// ✅ prefer toSignal — no async pipe needed
products = toSignal(this.productService.getAll(), { initialValue: [] });

// still valid for one-off
products$ = this.productService.getAll();
```
```html
<!-- with toSignal -->
@for (p of products(); track p.id) { ... }

<!-- with async pipe (still valid) -->
@for (p of products$ | async; track p.id) { ... }
```

---

## Reactive forms (typed)

```typescript
export class ProductFormComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    name:        ['', [Validators.required, Validators.maxLength(100)]],
    price:       [0,  [Validators.required, Validators.min(0)]],
    categoryId:  [null as number | null, Validators.required],
  });

  // Typed access
  get nameControl() { return this.form.controls.name; }

  submit() {
    if (this.form.invalid) return;
    const command: CreateProductCommand = this.form.getRawValue();
    // dispatch to store or call service
  }
}
```

---

## Directives (standalone)

```typescript
@Directive({
  selector: '[appHighlight]',
  standalone: true,
})
export class HighlightDirective {
  color = input('yellow');
  private el = inject(ElementRef);

  constructor() {
    effect(() => {
      this.el.nativeElement.style.backgroundColor = this.color();
    });
  }
}
```

---

## Lifecycle with signals

```typescript
export class DataComponent implements OnInit {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.someService.stream$.pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(data => this.localSignal.set(data));
  }
}
```

> **Prefer `effect()`** over `ngOnInit` for signal-reactive side effects when possible.

---

## OnPush + Signals checklist

- ✅ `changeDetection: ChangeDetectionStrategy.OnPush` on all components
- ✅ All mutable state in `signal()` / `computed()`
- ✅ Inputs via `input()` API
- ✅ No direct DOM mutation outside of `effect()`
- ❌ Never use `ChangeDetectorRef.detectChanges()` manually with Signals
