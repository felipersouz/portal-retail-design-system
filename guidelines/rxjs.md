# RxJS Guidelines

**Version:** RxJS 7.8 (included with Angular 20)

The golden rule: **prefer Signals for local/UI state, use RxJS for async streams and event composition.**

---

## Signals vs RxJS — When to Use Each

| Situation | Use |
|---|---|
| Local component state (loading, selected item, form mode) | `signal()` |
| Derived/computed values from state | `computed()` |
| HTTP requests in services | `Observable` (RxJS) |
| Search input with debounce | RxJS (`debounceTime` + `switchMap`) |
| Multiple parallel HTTP requests | RxJS (`forkJoin`) |
| Real-time / WebSocket streams | RxJS |
| Combining multiple observables | RxJS (`combineLatest`) |
| Observable displayed only in the template, no signal needed | `async` pipe |
| Observable needs to feed into `computed()` or other signals | `toSignal()` |
| Observable used in template AND in TypeScript logic | `toSignal()` |

---

## Subscription Management — Always use `takeUntilDestroyed`

Never manually unsubscribe with `ngOnDestroy`. Use `takeUntilDestroyed` from `@angular/core/rxjs-interop`.

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { DestroyRef, inject } from '@angular/core';

@Component({ ... })
export class MyComponent {
  // Option 1: inject DestroyRef and pass to takeUntilDestroyed
  private destroyRef = inject(DestroyRef);

  ngOnInit(): void {
    this.service.getData()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(data => this.items.set(data));
  }
}
```

```typescript
// Option 2: call takeUntilDestroyed() directly in field initializer (Angular 16+)
// Works outside ngOnInit — inject context is available
export class MyComponent {
  private data$ = this.service.getData().pipe(
    takeUntilDestroyed()  // ← no argument needed in injection context
  );
}
```

### ❌ Never do this

```typescript
// ❌ Never store and manually unsubscribe
private sub!: Subscription;

ngOnInit() { this.sub = obs$.subscribe(...); }
ngOnDestroy() { this.sub.unsubscribe(); }

// ❌ Never use take(1) as a lazy workaround for unsubscription
// take(1) is for "emit once then complete", not for cleanup
```

---

## Converting Observables to Signals — `toSignal`

Use `toSignal()` when you want to use an Observable value in a template reactively, without `async` pipe.

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

@Component({ ... })
export class ProductListComponent {
  private service = inject(ProductService);

  // ✅ Observable → Signal: auto-unsubscribes, no async pipe needed
  products = toSignal(this.service.getAll(), { initialValue: [] as Product[] });

  // ✅ With loading state
  private request$ = this.service.getAll();
  products = toSignal(this.request$, { initialValue: [] as Product[] });
}
```

```html
<!-- Template: use as signal, no async pipe -->
@for (product of products(); track product.id) {
  <span>{{ product.name }}</span>
}
```

---

## Converting Signals to Observables — `toObservable`

Use when you need to react to a signal change with RxJS operators (e.g., trigger HTTP on filter change).

```typescript
import { toObservable } from '@angular/core/rxjs-interop';

@Component({ ... })
export class ProductListComponent {
  searchTerm = signal('');

  // ✅ Signal → Observable → debounce → HTTP
  private results$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    distinctUntilChanged(),
    switchMap(term => this.service.search(term)),
    takeUntilDestroyed(),
  );

  results = toSignal(this.results$, { initialValue: [] as Product[] });
}
```

---

## The Right Flattening Operator

Choosing the wrong operator here causes bugs (duplicate requests, race conditions, memory leaks).

| Operator | Behavior | Use when |
|---|---|---|
| `switchMap` | Cancels previous, subscribes to new | Search, filter, navigation — only the **latest** matters |
| `concatMap` | Queues, waits for each to complete | Sequential tasks where **order matters** |
| `mergeMap` | All run in parallel, no cancellation | Fire-and-forget operations where **all must complete** |
| `exhaustMap` | Ignores new while one is in progress | Form submit — prevent **double submission** |

```typescript
// ✅ switchMap — search: cancel previous if user types again
searchTerm$.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.service.search(term))  // previous request cancelled
)

// ✅ exhaustMap — form submit: ignore clicks while saving
fromEvent(submitBtn, 'click').pipe(
  exhaustMap(() => this.service.save(this.form.value))  // ignores extra clicks
)

// ✅ concatMap — sequential uploads
filesToUpload$.pipe(
  concatMap(file => this.service.upload(file))  // one at a time, in order
)

// ✅ mergeMap — parallel requests with no ordering concern
ids$.pipe(
  mergeMap(id => this.service.getById(id))  // all parallel
)
```

---

## Parallel Requests — `forkJoin`

Use `forkJoin` to fire multiple requests simultaneously and wait for all to complete.

```typescript
// ✅ Load multiple resources at once (page init)
ngOnInit(): void {
  forkJoin({
    products:   this.productService.getAll(),
    categories: this.categoryService.getAll(),
    suppliers:  this.supplierService.getAll(),
  })
  .pipe(takeUntilDestroyed(this.destroyRef))
  .subscribe(({ products, categories, suppliers }) => {
    this.products.set(products);
    this.categories.set(categories);
    this.suppliers.set(suppliers);
    this.isLoading.set(false);
  });
}
```

> ⚠️ `forkJoin` requires ALL observables to complete. Use only with HTTP requests (they complete after one emission). Do not use with infinite streams.

---

## Reacting to Combined State — `combineLatest`

Use when you need the latest value of **multiple ongoing streams** together.

```typescript
// ✅ Re-filter whenever search OR status changes
combineLatest([
  this.searchTerm$,
  this.statusFilter$,
]).pipe(
  debounceTime(200),
  switchMap(([term, status]) => this.service.getAll({ term, status })),
  takeUntilDestroyed(this.destroyRef)
).subscribe(data => this.items.set(data));
```

---

## Error Handling

```typescript
// ✅ catchError: recover gracefully, return fallback
this.service.getAll().pipe(
  catchError(err => {
    console.error(err);
    this.hasError.set(true);
    return of([]);  // return safe empty value
  })
)

// ✅ retry: auto-retry on transient failures
this.service.getAll().pipe(
  retry({ count: 2, delay: 1000 })  // retry 2x with 1s delay
)

// ✅ finalize: always run (like finally) — great for loading state
this.service.getAll().pipe(
  finalize(() => this.isLoading.set(false))
)
```

---

## `async` Pipe — Template Subscription

The `async` pipe subscribes to an Observable (or Promise) in the template and **automatically unsubscribes** when the component is destroyed. It works natively with `ChangeDetectionStrategy.OnPush`.

### When to use `async` pipe vs `toSignal()`

| Scenario | Use |
|---|---|
| Observable only consumed in the template | `async` pipe |
| Observable needs to combine with `computed()` or signals | `toSignal()` |
| Observable value needed in TypeScript methods too | `toSignal()` |
| Streaming data (WebSocket, SSE) displayed in template | `async` pipe |
| You need `null` / `undefined` while loading (not an initial value) | `async` pipe |
| You want to avoid passing `initialValue` to `toSignal()` | `async` pipe |

---

### Basic Usage

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @if (products$ | async; as products) {
      @for (p of products; track p.id) {
        <span>{{ p.name }}</span>
      }
    }
  `
})
export class ProductListComponent {
  products$ = this.service.getAll();  // Observable stays in the class
  private service = inject(ProductService);
}
```

---

### Pattern: Loading + Error state with `async`

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [AsyncPipe],
  template: `
    @if (vm$ | async; as vm) {
      @if (vm.isLoading) {
        <p-progress-spinner />
      } @else if (vm.error) {
        <p-message severity="error" [text]="vm.error" />
      } @else {
        <eretail-dynamic-table [tableData]="vm.items" [tableColumns]="columns" />
      }
    }
  `
})
export class ProductListComponent {
  private service = inject(ProductService);

  // ✅ Single Observable with a view model (vm) — one async pipe in template
  vm$ = this.service.getAll().pipe(
    map(items => ({ items, isLoading: false, error: null as string | null })),
    startWith({ items: [], isLoading: true, error: null }),
    catchError(err => of({ items: [], isLoading: false, error: err.message })),
  );
}
```

---

### ✅ Always use `as` alias to avoid multiple subscriptions

```html
<!-- ✅ One subscription, value aliased to "products" -->
@if (products$ | async; as products) {
  <span>Total: {{ products.length }}</span>
  @for (p of products; track p.id) { ... }
}

<!-- ❌ WRONG: two subscriptions to the same observable -->
<span>Total: {{ (products$ | async)?.length }}</span>
@for (p of (products$ | async) ?? []; track p.id) { ... }
```

---

### Pattern: `async` with `@ngrx/store` selectors

```typescript
@Component({ ... })
export class OrderListComponent {
  private store = inject(Store);

  // Selectors are Observables — async pipe is natural here
  orders$        = this.store.select(selectAllOrders);
  isLoading$     = this.store.select(selectOrdersLoading);
  selectedOrder$ = this.store.select(selectSelectedOrder);
}
```

```html
@if (isLoading$ | async) {
  <p-progress-spinner />
}

@if (orders$ | async; as orders) {
  <eretail-dynamic-table [tableData]="orders" [tableColumns]="columns" />
}
```

---

### `async` pipe with `combineLatest` — avoid multiple pipes

```typescript
@Component({ ... })
export class ReportComponent {
  private productService = inject(ProductService);
  private categoryService = inject(CategoryService);

  // ✅ Combine into a single vm$ → single async pipe in template
  vm$ = combineLatest({
    products:   this.productService.getAll(),
    categories: this.categoryService.getAll(),
  }).pipe(
    map(({ products, categories }) => ({ products, categories, isLoading: false })),
    startWith({ products: [], categories: [], isLoading: true }),
    catchError(() => of({ products: [], categories: [], isLoading: false })),
  );
}
```

```html
@if (vm$ | async; as vm) {
  @if (vm.isLoading) {
    <p-progress-spinner />
  } @else {
    <!-- access vm.products and vm.categories here -->
  }
}
```

---

### Important: always import `AsyncPipe`

```typescript
import { AsyncPipe } from '@angular/common';

@Component({
  imports: [AsyncPipe, ...],
})
```

---

## Search / Autocomplete Pattern

```typescript
@Component({ ... })
export class SearchComponent {
  searchControl = new FormControl('');

  results = toSignal(
    this.searchControl.valueChanges.pipe(
      debounceTime(300),           // wait 300ms after user stops typing
      distinctUntilChanged(),      // skip if value didn't change
      filter(term => (term?.length ?? 0) >= 2),  // min 2 chars
      switchMap(term => this.service.search(term ?? '')),
      catchError(() => of([])),
      takeUntilDestroyed(),
    ),
    { initialValue: [] as SearchResult[] }
  );
}
```

---

## Rules

- ✅ Use `takeUntilDestroyed()` for every manual `.subscribe()` in a component
- ✅ Use `async` pipe for Observables consumed only in the template — it auto-unsubscribes
- ✅ Always use `as` alias with `async` pipe — never pipe the same observable twice
- ✅ Always import `AsyncPipe` from `@angular/common` in standalone components
- ✅ Combine multiple Observables into a single `vm$` with `combineLatest` — one `async` pipe
- ✅ Use `toSignal()` when the Observable value is also needed in TypeScript logic or `computed()`
- ✅ Use `toObservable()` to pipe signal changes into RxJS operators
- ✅ Use `startWith()` + `catchError()` + `map()` for the loading/error/data view model pattern
- ✅ Use `switchMap` for search/filter (latest wins)
- ✅ Use `exhaustMap` for form submit (prevent double-submission)
- ✅ Use `forkJoin` for parallel page-load requests
- ✅ Use `finalize()` to reset loading states
- ✅ Use `catchError()` with a safe fallback value
- ❌ Never use the same `async` pipe twice for the same observable — use `as` alias
- ❌ Never nest `.subscribe()` inside another `.subscribe()` — compose with operators
- ❌ Never use `ngOnDestroy` + manual `unsubscribe()` — use `takeUntilDestroyed`
- ❌ Never use `BehaviorSubject` for local component state — use `signal()`
- ❌ Never use `tap()` to mutate external state — use `subscribe()` or `toSignal()`
