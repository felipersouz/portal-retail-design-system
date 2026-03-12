# Angular Modern Features (v17–v20)

This document covers Angular features introduced from v17 onwards that are required or preferred in this project.
Always use these over legacy equivalents.

---

## Signals — Core State Primitives

### `signal()` — writable state

```typescript
// Basic signal
count = signal(0);
items = signal<Product[]>([]);
isLoading = signal(false);

// Reading
console.log(this.count());   // call as function

// Writing
this.count.set(5);
this.count.update(n => n + 1);
this.items.set(newList);
this.items.update(prev => [...prev, newItem]);
```

### `computed()` — derived state (memoized)

```typescript
// Automatically recalculates when dependencies change
totalItems = computed(() => this.items().length);
activeItems = computed(() => this.items().filter(i => i.active));
pageTitle = computed(() => this.isEditMode() ? 'Edit' : 'New');

// Can depend on multiple signals
summary = computed(() => ({
  total: this.items().length,
  active: this.items().filter(i => i.active).length,
  loading: this.isLoading(),
}));
```

### `linkedSignal()` — derived writable state (Angular 19+)

Use when you need a signal that resets when a source changes but can also be manually overridden.

```typescript
import { linkedSignal } from '@angular/core';

// Resets selectedId whenever items changes
items = signal<Product[]>([]);
selectedId = linkedSignal(() => this.items()[0]?.id ?? null);

// User can still manually set it
onSelect(id: number) {
  this.selectedId.set(id);  // overrides the linked value
}
// But when items changes → selectedId resets to items[0].id automatically
```

### `effect()` — side effects on signal changes

```typescript
import { effect } from '@angular/core';

constructor() {
  // Runs whenever selectedId changes
  effect(() => {
    const id = this.selectedId();
    if (id) this.loadDetail(id);
  });
}
```

> ⚠️ **effect() rules:**
> - Only create effects in constructor or field initializer
> - Never update signals inside `effect()` by default — use `allowSignalWrites: true` only when unavoidable
> - Prefer `computed()` over `effect()` for derived values

---

## `resource()` — Async Data Loading (Angular 19+)

`resource()` replaces the `isLoading + subscribe + set` pattern for async data. It manages loading, error, and value states automatically.

```typescript
import { resource } from '@angular/core';

@Component({ ... })
export class ProductListComponent {
  private service = inject(ProductService);

  // Automatically fetches on init and when selectedId changes
  selectedId = signal<number | null>(null);

  productResource = resource({
    request: () => this.selectedId(),          // reactive dependency
    loader: ({ request: id }) => {
      if (!id) return Promise.resolve(null);
      return firstValueFrom(this.service.getById(id));
    },
  });

  // Access states in template:
  // productResource.value()    → the data (or undefined)
  // productResource.isLoading() → boolean
  // productResource.error()    → error if any
  // productResource.status()   → 'idle' | 'loading' | 'resolved' | 'error'
}
```

```html
@if (productResource.isLoading()) {
  <p-progress-spinner />
} @else if (productResource.error()) {
  <p-message severity="error" text="Failed to load product" />
} @else {
  <span>{{ productResource.value()?.name }}</span>
}
```

### `resource()` for list pages

```typescript
filterSignal = signal({ status: 'active', page: 1 });

productsResource = resource({
  request: () => this.filterSignal(),
  loader: ({ request: filter }) =>
    firstValueFrom(this.service.getAll(filter)),
});

// Refresh manually
refresh(): void {
  this.productsResource.reload();
}

// Change filter → automatically reloads
applyFilter(status: string): void {
  this.filterSignal.update(f => ({ ...f, status }));
}
```

---

## Signal-based Queries — `viewChild`, `contentChild`

Replace `@ViewChild` and `@ContentChild` decorators with signal-based functions.

```typescript
import { viewChild, viewChildren, contentChild } from '@angular/core';
import { ElementRef } from '@angular/core';

@Component({ ... })
export class MyComponent {
  // ✅ Signal-based (Angular 17.2+)
  searchInput = viewChild<ElementRef>('searchInput');
  searchInputRequired = viewChild.required<ElementRef>('searchInput');  // throws if not found
  listItems = viewChildren<ElementRef>('item');

  // Access the value
  focusInput(): void {
    this.searchInput()?.nativeElement.focus();
  }
}
```

```typescript
// ❌ Old decorator-based (avoid in new code)
@ViewChild('searchInput') searchInput!: ElementRef;
@ViewChildren('item') items!: QueryList<ElementRef>;
```

---

## `model()` — Two-way Bindable Signal Input

Use `model()` when a child component needs to expose a value that the parent can both read and write (replaces `@Input() + @Output()` pairs for two-way binding).

```typescript
import { model } from '@angular/core';

// Child component
@Component({
  selector: 'app-quantity-input',
  template: `
    <p-inputnumber
      [value]="quantity()"
      (valueChange)="quantity.set($event)" />
  `
})
export class QuantityInputComponent {
  quantity = model<number>(1);  // writable + emits changes to parent
}
```

```html
<!-- Parent: two-way binding with [(quantity)] -->
<app-quantity-input [(quantity)]="orderQuantity" />
```

---

## `inject()` in Any Context

`inject()` works in field initializers, constructor, and any injection context — not just the constructor parameter list.

```typescript
@Component({ ... })
export class MyComponent {
  // ✅ Field initializer — most common pattern
  private service = inject(ProductService);
  private router = inject(Router);
  private destroyRef = inject(DestroyRef);

  // ✅ In factory functions / providers
  static provide(): Provider {
    return {
      provide: MY_TOKEN,
      useFactory: () => inject(ConfigService).get('value'),
    };
  }
}
```

---

## `DestroyRef` — Lifecycle Cleanup without `ngOnDestroy`

```typescript
import { DestroyRef, inject } from '@angular/core';

@Component({ ... })
export class MyComponent {
  private destroyRef = inject(DestroyRef);

  constructor() {
    // Register cleanup without implementing ngOnDestroy
    this.destroyRef.onDestroy(() => {
      console.log('Component destroyed, cleanup done');
      this.someExternalSubscription.unsubscribe();
    });
  }
}
```

---

## `afterRender` / `afterNextRender` — DOM Access

Replace `ngAfterViewInit` for DOM operations that need to happen after rendering.

```typescript
import { afterNextRender, afterRender } from '@angular/core';

@Component({ ... })
export class ChartComponent {
  constructor() {
    // Runs once after the FIRST render — use for init
    afterNextRender(() => {
      this.initChart();  // safe to access DOM here
    });

    // Runs after EVERY render — use with caution
    afterRender(() => {
      this.updateChartSize();
    });
  }
}
```

---

## `@defer` — Deferred Loading of Heavy Components

Use `@defer` for heavy components (charts, maps, rich text editors) to improve initial load performance.

```html
<!-- Loads TinyMCE only when it enters the viewport -->
@defer (on viewport) {
  <tinymce-editor [formControl]="contentControl" />
} @placeholder {
  <div class="p-4 text-muted">Loading editor...</div>
} @loading (minimum 200ms) {
  <p-progress-spinner />
} @error {
  <p-message severity="error" text="Failed to load editor" />
}

<!-- Loads chart on user interaction -->
@defer (on interaction) {
  <app-chart [data]="chartData()" />
} @placeholder {
  <div class="chart-placeholder">Click to load chart</div>
}
```

### Defer triggers

| Trigger | When |
|---|---|
| `on idle` | Browser is idle (default) |
| `on viewport` | Element enters the viewport |
| `on interaction` | User clicks or focuses |
| `on hover` | User hovers over placeholder |
| `on timer(2s)` | After a delay |
| `when condition` | When a boolean expression becomes true |

---

## Native Control Flow (`@if`, `@for`, `@switch`)

Always use native control flow. Never use structural directives.

```html
<!-- @if / @else -->
@if (isLoading()) {
  <p-progress-spinner />
} @else if (hasError()) {
  <p-message severity="error" text="Error loading data" />
} @else {
  <eretail-dynamic-table [tableData]="items()" [tableColumns]="columns" />
}

<!-- @for — track is REQUIRED -->
@for (item of items(); track item.id) {
  <app-stat-card [title]="item.name" [stats]="item.stats" />
} @empty {
  <p class="text-muted">No items found.</p>
}

<!-- @switch -->
@switch (user().role) {
  @case ('admin')   { <app-admin-panel /> }
  @case ('manager') { <app-manager-panel /> }
  @default          { <app-viewer-panel /> }
}
```

---

## Rules

- ✅ Use `signal()` / `computed()` for all component state
- ✅ Use `linkedSignal()` for derived writable state that resets on source change
- ✅ Use `resource()` for async data loading (replaces loading signal + subscribe pattern)
- ✅ Use `viewChild()` / `viewChildren()` instead of `@ViewChild` / `@ViewChildren`
- ✅ Use `model()` for two-way bindable inputs
- ✅ Use `inject()` in field initializers — never constructor parameters
- ✅ Use `DestroyRef` + `takeUntilDestroyed()` instead of `ngOnDestroy`
- ✅ Use `afterNextRender()` for one-time DOM init instead of `ngAfterViewInit`
- ✅ Use `@defer` for heavy components (maps, charts, rich text)
- ✅ Use `@for` with `track` always — never omit track expression
- ❌ Never use `@ViewChild` / `@ContentChild` decorators in new code
- ❌ Never use `ngOnDestroy` just for unsubscribing — use `takeUntilDestroyed`
- ❌ Never use `*ngIf` / `*ngFor` — use `@if` / `@for`
