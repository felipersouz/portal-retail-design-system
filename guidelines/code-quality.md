# Code Quality Guidelines

Standards for writing clean, maintainable, and type-safe code in portal-retail-frontend.

---

## TypeScript Strictness

The project uses `strict: true`. These rules are enforced by the compiler:

```typescript
// ✅ Explicit return types on public methods
getById(id: number): Observable<Product> { ... }

// ✅ Use unknown instead of any
function parseResponse(data: unknown): Product {
  if (!isProduct(data)) throw new Error('Invalid data');
  return data;
}

// ✅ Non-null assertion only when you're 100% sure — prefer optional chaining
const name = user?.profile?.name ?? 'Anonymous';

// ✅ Type narrowing instead of casting
if (error instanceof HttpErrorResponse) {
  console.error(error.status);
}

// ❌ Never use any
function process(data: any) { ... }  // WRONG

// ❌ Never suppress type errors with // @ts-ignore
// @ts-ignore  ← FORBIDDEN
```

---

## Naming Conventions

### Variables and Functions

```typescript
// ✅ Observables: suffix with $
const products$ = this.service.getAll();
const search$ = this.searchControl.valueChanges;

// ✅ Signals: no suffix (they ARE the state)
items = signal<Product[]>([]);
isLoading = signal(false);

// ✅ Booleans: prefix with is/has/can/should
isLoading = signal(false);
hasError = signal(false);
canDelete = computed(() => this.user().role === 'admin');
shouldRefresh = false;

// ✅ Arrays: plural nouns
products = signal<Product[]>([]);
selectedIds = signal<number[]>([]);

// ✅ Private methods/fields: no underscore — TypeScript access modifiers are enough
private service = inject(ProductService);
private loadData(): void { ... }

// ❌ Avoid generic names
const data = ...;   // What data? Use: products, users, report
const temp = ...;   // Always name by intent
const x = ...;      // Never single-letter vars outside loops
```

### Interfaces and Types

```typescript
// ✅ Interfaces for data shapes (API responses, entities)
interface Product {
  id: number;
  name: string;
  status: ProductStatus;
}

// ✅ DTOs for request payloads
interface CreateProductDto {
  name: string;
  price: number;
  categoryId: number;
}

interface UpdateProductDto extends Partial<CreateProductDto> {}

// ✅ Types for unions, mapped types, utilities
type ProductStatus = 'active' | 'inactive' | 'draft';
type ProductMap = Record<number, Product>;

// ❌ Never prefix interfaces with I
interface IProduct { ... }  // WRONG — use Product
```

---

## Function Design

### Single Responsibility

```typescript
// ✅ Each method does ONE thing
private loadProducts(): void {
  this.isLoading.set(true);
  this.productService.getAll()
    .pipe(finalize(() => this.isLoading.set(false)), takeUntilDestroyed(this.destroyRef))
    .subscribe(data => this.products.set(data));
}

private handleSaveSuccess(): void {
  this.isSaving.set(false);
  this.router.navigate(['/products']);
}

private handleSaveError(err: HttpErrorResponse): void {
  this.isSaving.set(false);
  console.error('[ProductForm] Save failed:', err);
}

// ❌ Avoid doing multiple things in one method
onSubmit(): void {
  // validating + saving + navigating + error handling + logging  ← too much
}
```

### Early Return (Guard Clauses)

```typescript
// ✅ Return early to avoid deep nesting
onSubmit(): void {
  if (this.form.invalid) {
    this.form.markAllAsTouched();
    return;
  }

  if (this.isSaving()) return;

  this.save();
}

// ❌ Deep nesting
onSubmit(): void {
  if (this.form.valid) {
    if (!this.isSaving()) {
      // actual logic buried deep
    }
  }
}
```

### Max Complexity

- **Max lines per method:** 20 (extract if longer)
- **Max parameters:** 3 (use an options object if more are needed)

```typescript
// ❌ Too many params
function buildTable(data, cols, buttons, filters, title, subtitle, paginate) {}

// ✅ Options object
interface TableConfig {
  data: any[];
  columns: EretailTableColumn[];
  buttons?: TableButton[];
  filters?: TableFilterField[];
  title?: string;
}
function buildTable(config: TableConfig) {}
```

---

## No Magic Values

```typescript
// ❌ Magic numbers / strings
if (user.role === 3) { ... }
setTimeout(() => reload(), 2000);
if (items.length > 50) showWarning();

// ✅ Named constants or enums
enum UserRole { Admin = 1, Manager = 2, Viewer = 3 }
const RELOAD_DELAY_MS = 2000;
const MAX_TABLE_ROWS_WARNING = 50;

if (user.role === UserRole.Viewer) { ... }
setTimeout(() => reload(), RELOAD_DELAY_MS);
if (items.length > MAX_TABLE_ROWS_WARNING) showWarning();
```

---

## Immutability

```typescript
// ✅ Never mutate arrays/objects directly — create new references
this.items.update(prev => [...prev, newItem]);
this.items.update(prev => prev.filter(i => i.id !== id));
this.items.update(prev =>
  prev.map(i => i.id === id ? { ...i, status: 'inactive' } : i)
);

// ❌ Never mutate in place
this.items().push(newItem);           // WRONG
this.items()[0].status = 'inactive';  // WRONG
```

---

## Error Handling in Components

```typescript
@Component({ ... })
export class ProductListComponent {
  hasError = signal(false);
  errorMessage = signal('');

  private loadProducts(): void {
    this.isLoading.set(true);
    this.productService.getAll().pipe(
      catchError((err: HttpErrorResponse) => {
        this.hasError.set(true);
        this.errorMessage.set(err.message);
        return of([]);
      }),
      finalize(() => this.isLoading.set(false)),
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(data => this.products.set(data));
  }
}
```

```html
@if (hasError()) {
  <p-message severity="error" [text]="errorMessage()" />
}
```

---

## Console Logs

```typescript
// ✅ Use structured prefix for easy filtering
console.error('[ProductService] Failed to load:', err);
console.warn('[AuthGuard] Unauthorized access to:', url);

// ❌ Never leave generic logs in production code
console.log('data', data);   // REMOVE before commit
console.log('here');         // REMOVE before commit
```

---

## Comments

```typescript
// ✅ Comment WHY, not WHAT (the code shows what)
// Bypass cache: the API returns stale data within the same session
const headers = new HttpHeaders({ 'Cache-Control': 'no-cache' });

// ❌ Useless comments that repeat the code
// Get all products
getAll(): Observable<Product[]> { ... }

// ❌ Commented-out code — delete it, git has history
// const old = this.service.getOldMethod();
```

---

## Import Order

Always follow this order (ESLint enforces it):

```typescript
// 1. Angular core
import { Component, inject, signal } from '@angular/core';
import { Router } from '@angular/router';

// 2. Third-party libraries
import { CardModule } from 'primeng/card';
import { SelectModule } from 'primeng/select';

// 3. Project aliases (@core, @shared)
import { ProductService } from '@core/services/product.service';
import { Product } from '@core/models/product.model';
import { ContentWrapperComponent } from '@shared/components/content-wrapper/content-wrapper.component';

// 4. Relative imports (avoid when alias exists)
import { someUtil } from './utils/helper';
```

---

## Rules Summary

- ✅ `strict: true` always — never disable compiler checks
- ✅ `unknown` instead of `any` when type is uncertain
- ✅ Observable variables suffixed with `$`
- ✅ Boolean signals/vars prefixed with `is` / `has` / `can`
- ✅ Early return (guard clauses) over deep nesting
- ✅ Named constants for all magic values
- ✅ Immutable state updates (spread, filter, map)
- ✅ Structured console logs with `[ClassName]` prefix
- ✅ Comments explain WHY, not WHAT
- ❌ Never use `any`
- ❌ Never commit `console.log` debug lines
- ❌ Never leave commented-out code
- ❌ Never prefix interfaces with `I`
