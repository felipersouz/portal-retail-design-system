# Angular Architecture Guidelines

**Framework:** Angular 20  
**Style:** Standalone Components, Signals, OnPush

---

## Project Stack

| Category | Technology | Notes |
|---|---|---|
| Framework | Angular 20 | Standalone components only |
| UI (primary) | PrimeNG 20 | **Always first choice** — tables, forms, dialogs, overlays, menus, navigation, inputs, buttons, tooltips, badges, chips, calendars, dropdowns |
| UI (fallback) | Angular Material 20 | Only when PrimeNG has no equivalent component |
| CSS Framework | Bootstrap 5 | Utility classes and `bi bi-*` icons only — never Bootstrap JS components |
| Icons | Bootstrap Icons | `bi bi-*` classes |
| Forms (dynamic) | ngx-formly 7 | Complex/dynamic form schemas |
| Forms (static) | Angular Reactive Forms | All other forms |
| Charts | ApexCharts + ng-apexcharts | All chart types |
| Maps | Leaflet + MapLibre GL | Geo features |
| Rich Text | TinyMCE 7 | Content editors |
| Real-time | SignalR (@microsoft/signalr) | Live updates |
| HTTP | Angular HttpClient | Services only |
| i18n | @ngx-translate | All user-facing strings |

---

## Component Rules

### ✅ Always

```typescript
@Component({
  selector: 'app-feature-name',
  // standalone: true  ← DO NOT add this, it's the default in Angular 17+
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [...],
  templateUrl: './feature-name.component.html',
  styleUrl: './feature-name.component.scss',
})
export class FeatureNameComponent {
  // Use signals for all state
  items = signal<Item[]>([]);
  isLoading = signal(false);

  // Use computed for derived state
  totalItems = computed(() => this.items().length);

  // Use inject() not constructor injection
  private service = inject(FeatureService);

  // Use input() not @Input()
  title = input.required<string>();
  subtitle = input<string>();

  // Use output() not @Output() + EventEmitter
  saved = output<Item>();
}
```

### ❌ Never

```typescript
// ❌ Don't use constructor injection
constructor(private service: FeatureService) {}

// ❌ Don't use @Input() / @Output()
@Input() title: string;
@Output() saved = new EventEmitter<Item>();

// ❌ Don't use @HostBinding / @HostListener
@HostBinding('class.active') isActive = true;

// ❌ Don't use ngClass / ngStyle
<div [ngClass]="{'active': isActive}">
<div [ngStyle]="{'color': 'red'}">

// ❌ Don't use *ngIf / *ngFor / *ngSwitch
<div *ngIf="show">
<li *ngFor="let item of items">
```

### ✅ Template Patterns

```html
<!-- Control flow -->
@if (isLoading()) {
  <p-progress-spinner />
} @else {
  <eretail-dynamic-table [tableData]="items()" [tableColumns]="columns" />
}

@for (item of items(); track item.id) {
  <app-stat-card [title]="item.name" [stats]="item.stats" />
}

@switch (status()) {
  @case ('active') { <span class="text-success">Active</span> }
  @case ('inactive') { <span class="text-danger">Inactive</span> }
  @default { <span>Unknown</span> }
}

<!-- Class bindings (not ngClass) -->
<div [class.active]="isActive()">
<div [class]="'card ' + colorClass()">

<!-- Style bindings (not ngStyle) -->
<div [style.color]="textColor()">
```

---

## State Management Rules

```typescript
// ✅ Local state → signals
items = signal<Product[]>([]);

// ✅ Derived state → computed
filteredItems = computed(() =>
  this.items().filter(p => p.active)
);

// ✅ Updating state
this.items.set(newItems);           // Full replacement
this.items.update(prev => [...prev, newItem]);  // Partial update

// ❌ Never use .mutate()
this.items.mutate(items => items.push(item));  // WRONG
```

---

## Service Rules

```typescript
@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private apiUrl = inject(API_URL_TOKEN); // or from environment

  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>(`${this.apiUrl}/products`);
  }

  getById(id: number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/products/${id}`);
  }

  create(product: CreateProductDto): Observable<Product> {
    return this.http.post<Product>(`${this.apiUrl}/products`, product);
  }

  update(id: number, product: UpdateProductDto): Observable<Product> {
    return this.http.put<Product>(`${this.apiUrl}/products/${id}`, product);
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/products/${id}`);
  }
}
```

---

## Routing Rules

```typescript
// All feature routes use lazy loading
export const APP_ROUTES: Routes = [
  {
    path: 'products',
    loadChildren: () =>
      import('./pages/product/product.routes').then(m => m.PRODUCT_ROUTES),
    canActivate: [authGuard],
  },
];

// Feature routes file: product.routes.ts
export const PRODUCT_ROUTES: Routes = [
  { path: '', component: ProductListComponent },
  { path: 'create', component: ProductFormComponent },
  { path: 'edit/:id', component: ProductFormComponent },
];
```

---

## Form Rules

### Reactive Forms (static forms)

```typescript
// ✅ Use FormBuilder via inject()
private fb = inject(FormBuilder);

form = this.fb.group({
  name: ['', [Validators.required, Validators.maxLength(100)]],
  email: ['', [Validators.required, Validators.email]],
  status: [null as string | null, Validators.required],
});

// ✅ Subscribe in template with async pipe, or use form.value directly in submit
onSubmit(): void {
  if (this.form.invalid) return;
  this.service.create(this.form.getRawValue()).subscribe(...);
}
```

### ngx-formly (dynamic forms)

Use `ngx-formly` when the form structure is data-driven or configured by the backend.

```typescript
fields: FormlyFieldConfig[] = [
  {
    key: 'name',
    type: 'input',
    props: { label: 'Name', required: true },
  },
];
```
