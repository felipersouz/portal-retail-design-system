# Pattern: Listing Page

The standard pattern for pages that list/query data from an API.
Uses skeleton loading, inline error state, `ToastService` for feedback,
and a reactive filter bar with auto-apply, debounce, and dependent selects.

## Structure

```
feature-list/
├── feature-list.component.ts    # Main component
├── feature-list.component.html  # Template
└── feature-list.component.scss  # Styles (usually empty for list pages)
```

---

## TypeScript Template — with filters

```typescript
import {
  Component, OnInit, inject, signal, computed,
  ChangeDetectionStrategy, DestroyRef
} from '@angular/core';
import { Router } from '@angular/router';
import { FormControl, FormGroup, ReactiveFormsModule } from '@angular/forms';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import {
  Subject, catchError, debounceTime, distinctUntilChanged,
  of, switchMap, tap
} from 'rxjs';
import { SkeletonModule } from 'primeng/skeleton';
import { CardModule } from 'primeng/card';
import { MessageModule } from 'primeng/message';
import { InputTextModule } from 'primeng/inputtext';
import { SelectModule } from 'primeng/select';
import { ButtonModule } from 'primeng/button';
import { Table, TableColumns, TableAction, ResponseData } from '@shared/components/table/table';
import { ContentWrapperComponent } from '@shared/components/content-wrapper/content-wrapper.component';
import { HeadingComponent } from '@shared/components/heading/heading.component';
import { CustomButtonComponent } from '@shared/components/button/button/button.component';
import { ToastService } from '@core/helpers/toast.service';
import { FeatureService } from '@core/services/feature.service';
import { Feature } from '@core/models/feature.model';
import { PaginationParams } from '@core/services/base.service';

// ── Local interfaces ───────────────────────────────────────────────────────
interface SelectOption {
  label: string;
  value: string | number;
}

// ── Component ─────────────────────────────────────────────────────────────
@Component({
  selector: 'app-feature-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    ReactiveFormsModule,
    ContentWrapperComponent,
    HeadingComponent,
    CustomButtonComponent,
    Table,
    SkeletonModule,
    CardModule,
    MessageModule,
    InputTextModule,
    SelectModule,
    ButtonModule,
  ],
  templateUrl: './feature-list.component.html',
})
export class FeatureListComponent implements OnInit {
  private service    = inject(FeatureService);
  private router     = inject(Router);
  private toast      = inject(ToastService);
  private destroyRef = inject(DestroyRef);

  // ── Table state ──────────────────────────────────────────────────────────
  data      = signal<ResponseData<Feature>>({ total: 0, items: [] });
  isLoading = signal(false);
  hasError  = signal(false);

  // Remembers the last pagination so filter changes reset to page 1
  // but keep the same pageSize and sort
  private lastPagination = signal<PaginationParams | null>(null);

  // Subject-based trigger — switchMap cancels in-flight requests automatically
  private load$ = new Subject<PaginationParams>();

  // ── Table columns ────────────────────────────────────────────────────────
  cols: TableColumns[] = [
    { field: 'name',         header: 'Name' },
    { field: 'categoryName', header: 'Category' },
    { field: 'statusName',   header: 'Status', type: 'badge', badgeSeverityMap: {
      Active:   'success',
      Inactive: 'danger',
      Pending:  'warn',
    }},
    { field: 'createdAt', header: 'Created at', type: 'date' },
  ];

  actions: TableAction[] = [
    {
      icon: 'bi bi-eye',
      buttonClass: 'info',
      action: (row: Feature) => this.router.navigate(['/feature/detail', row.id]),
    },
    {
      icon: 'bi bi-pencil',
      buttonClass: 'secondary',
      action: (row: Feature) => this.router.navigate(['/feature/edit', row.id]),
    },
  ];

  // ── Filter form ──────────────────────────────────────────────────────────
  filters = new FormGroup({
    searchTerm:   new FormControl<string>(''),
    categoryName: new FormControl<string>(''),
    type:         new FormControl<string | null>(null),
    status:       new FormControl<number | null>(null),
  });

  // Static select options
  typeOptions: SelectOption[] = [
    { label: 'Type A', value: 'A' },
    { label: 'Type B', value: 'B' },
  ];

  // Dynamic options: populated by a dependent filter (type -> status)
  statusOptions = signal<SelectOption[]>([]);

  // Status options per type — drives the dependent filter
  private readonly statusOptionsByType: Record<string, SelectOption[]> = {
    A: [
      { label: 'Active',  value: 1 },
      { label: 'Pending', value: 2 },
    ],
    B: [
      { label: 'Active',   value: 1 },
      { label: 'Inactive', value: 2 },
      { label: 'Archived', value: 3 },
    ],
  };

  // Disables the status select until a type is chosen
  isTypeSelected = computed(() => Boolean(this.filters.controls.type.value));

  ngOnInit(): void {
    this.listenForLoads();
    this.listenForDependentFilter();
    this.listenForFilterChanges();
  }

  // Called by <app-table> (pageChange output) on page/sort changes
  onPageChange(params: PaginationParams): void {
    this.lastPagination.set(params);
    this.load$.next(this.buildParams(params));
  }

  // Resets to page 1, keeps current pageSize and sort
  applyFilters(): void {
    const last = this.lastPagination();
    const params: PaginationParams = {
      page: 1,
      pageSize:   last?.pageSize   ?? 15,
      sortColumn: last?.sortColumn,
      sortOrder:  last?.sortOrder,
    };
    this.load$.next(this.buildParams(params));
  }

  clearFilters(): void {
    this.filters.reset({
      searchTerm:   '',
      categoryName: '',
      type:         null,
      status:       null,
    });
    this.applyFilters();
  }

  onRetry(): void {
    this.applyFilters();
  }

  // ── Private ──────────────────────────────────────────────────────────────

  // Merges pagination params with current filter values.
  // Use || undefined for strings and ?? undefined for numbers/nullables
  // so empty values are stripped from the request.
  private buildParams(params: PaginationParams): PaginationParams {
    const f = this.filters.getRawValue();
    return {
      ...params,
      searchTerm:   f.searchTerm   || undefined,
      categoryName: f.categoryName || undefined,
      type:         f.type         ?? undefined,
      status:       f.status       ?? undefined,
    };
  }

  // switchMap cancels the previous request if a new one arrives while loading
  private listenForLoads(): void {
    this.load$.pipe(
      tap(() => {
        this.isLoading.set(true);
        this.hasError.set(false);
      }),
      switchMap((params) =>
        this.service.getAll(params).pipe(
          catchError(() => {
            this.hasError.set(true);
            this.toast.showErrorToast('Error', 'Failed to load features. Please try again.');
            return of({ total: 0, items: [] });
          })
        )
      ),
      takeUntilDestroyed(this.destroyRef),
    ).subscribe((response) => {
      this.isLoading.set(false);
      this.data.set(response);
    });
  }

  // Auto-apply on every filter change: debounce prevents excess API calls.
  // distinctUntilChanged prevents re-fetching when the value did not really change.
  private listenForFilterChanges(): void {
    this.filters.valueChanges.pipe(
      debounceTime(800),
      distinctUntilChanged((prev, next) => JSON.stringify(prev) === JSON.stringify(next)),
      takeUntilDestroyed(this.destroyRef),
    ).subscribe(() => this.applyFilters());
  }

  // Dependent filter: when "type" changes, update statusOptions
  // and reset "status" without triggering a redundant API call.
  private listenForDependentFilter(): void {
    this.filters.controls.type.valueChanges.pipe(
      takeUntilDestroyed(this.destroyRef),
    ).subscribe((type) => {
      this.statusOptions.set(type ? this.statusOptionsByType[type] ?? [] : []);
      this.filters.controls.status.setValue(null, { emitEvent: false });
    });
  }
}
```

---

## HTML Template — with filter bar

```html
<content-wrapper>
  <div class="d-flex justify-content-between align-items-center mb-3">
    <app-heading title="Feature List" subtitle="Manage all features" />

    <app-button
      buttonClass="bt-primary"
      icon="plus"
      routerLink="/feature/create"
      data-cy="feature-new-btn">
      New Feature
    </app-button>
  </div>

  <!-- Filter bar -->
  <p-card styleClass="mb-3">
    <div class="row g-2" [formGroup]="filters">

      <!-- Free text search -->
      <div class="col-md-4">
        <input
          pInputText
          id="searchTerm"
          formControlName="searchTerm"
          placeholder="Search..."
          class="p-inputtext-sm w-100"
          data-cy="feature-search-input" />
      </div>

      <!-- Additional text filter -->
      <div class="col-md-4">
        <input
          pInputText
          id="categoryName"
          formControlName="categoryName"
          placeholder="Category"
          class="p-inputtext-sm w-100"
          data-cy="feature-category-input" />
      </div>

      <!-- Select: drives a dependent filter -->
      <div class="col-md-4">
        <p-select
          inputId="type"
          formControlName="type"
          [options]="typeOptions"
          optionLabel="label"
          optionValue="value"
          placeholder="Type"
          [showClear]="true"
          size="small"
          class="w-100"
          [inputAttrs]="{ 'data-cy': 'feature-type-select' }" />
      </div>

      <!-- Dependent select: disabled until "type" is selected -->
      <div class="col-md-4">
        <p-select
          inputId="status"
          formControlName="status"
          [options]="statusOptions()"
          optionLabel="label"
          optionValue="value"
          placeholder="Status"
          [showClear]="true"
          [disabled]="!isTypeSelected()"
          size="small"
          class="w-100"
          [inputAttrs]="{ 'data-cy': 'feature-status-select' }" />
      </div>

      <!-- Clear filters button -->
      <div class="col-md-4 d-flex align-items-end">
        <p-button
          icon="pi pi-filter-slash"
          label="Clear filters"
          severity="secondary"
          [outlined]="true"
          size="small"
          styleClass="w-100"
          data-cy="feature-clear-filters-btn"
          (onClick)="clearFilters()" />
      </div>

    </div>
  </p-card>

  <!-- Loading state: skeleton -->
  @if (isLoading()) {
    <p-card>
      <p-skeleton width="40%" height="1.25rem" styleClass="mb-4" />
      @for (_ of [1,2,3,4,5,6]; track $index) {
        <p-skeleton width="100%" height="2.5rem" styleClass="mb-2" />
      }
    </p-card>
  }

  <!-- Error state: inline message with retry -->
  @else if (hasError()) {
    <p-card>
      <div class="d-flex flex-column align-items-center gap-3 py-4">
        <p-message
          severity="error"
          text="Failed to load the data. Check your connection and try again." />
        <app-button
          buttonClass="bt-default"
          icon="arrow-clockwise"
          data-cy="feature-retry-btn"
          (clicked)="onRetry()">
          Try again
        </app-button>
      </div>
    </p-card>
  }

  <!-- Success state: data table -->
  @else {
    <app-table
      [cols]="cols"
      [data]="data()"
      [actions]="actions"
      [rowPerPageOptions]="[15, 25, 50]"
      (pageChange)="onPageChange($event)" />
  }
</content-wrapper>
```

---

## Filter Patterns Reference

### 1 — Auto-apply with debounce (no "Search" button)

Fires the API call automatically after the user stops typing for 800 ms.
`distinctUntilChanged` prevents re-fetching when the value did not change (e.g. blur without edit).

```typescript
private listenForFilterChanges(): void {
  this.filters.valueChanges.pipe(
    debounceTime(800),
    distinctUntilChanged((prev, next) => JSON.stringify(prev) === JSON.stringify(next)),
    takeUntilDestroyed(this.destroyRef),
  ).subscribe(() => this.applyFilters());
}
```

### 2 — Subject + switchMap (cancels in-flight requests)

Each new emission through `load$` cancels the previous in-flight HTTP call.
This prevents stale results from overwriting fresh ones (race condition).

```typescript
private load$ = new Subject<PaginationParams>();

private listenForLoads(): void {
  this.load$.pipe(
    tap(() => this.isLoading.set(true)),
    switchMap((params) =>
      this.service.getAll(params).pipe(
        catchError(() => {
          this.hasError.set(true);
          this.toast.showErrorToast('Error', 'Failed to load data.');
          return of({ total: 0, items: [] });
        })
      )
    ),
    takeUntilDestroyed(this.destroyRef),
  ).subscribe((response) => {
    this.isLoading.set(false);
    this.data.set(response);
  });
}
```

### 3 — Dependent filter (select A drives select B)

When the parent select changes:
1. Update the child options from a local lookup map.
2. Reset the child value with `{ emitEvent: false }` to avoid triggering `valueChanges` again.
3. Disable the child select in the template until the parent has a value.

```typescript
private readonly statusOptionsByType: Record<string, SelectOption[]> = {
  A: [{ label: 'Active', value: 1 }, { label: 'Pending', value: 2 }],
  B: [{ label: 'Active', value: 1 }, { label: 'Archived', value: 3 }],
};

statusOptions = signal<SelectOption[]>([]);

private listenForDependentFilter(): void {
  this.filters.controls.type.valueChanges.pipe(
    takeUntilDestroyed(this.destroyRef),
  ).subscribe((type) => {
    this.statusOptions.set(type ? this.statusOptionsByType[type] ?? [] : []);
    this.filters.controls.status.setValue(null, { emitEvent: false }); // no extra API call
  });
}
```

```html
<!-- Disabled until parent has a value -->
<p-select
  formControlName="status"
  [options]="statusOptions()"
  [disabled]="!isTypeSelected()" />
```

### 4 — buildParams (merges pagination + filters cleanly)

Pagination and filter concerns stay separate until the last moment.
`|| undefined` strips empty strings; `?? undefined` strips null/undefined numbers.

```typescript
private buildParams(params: PaginationParams): PaginationParams {
  const f = this.filters.getRawValue();
  return {
    ...params,
    searchTerm: f.searchTerm || undefined,   // '' -> omitted
    status:     f.status     ?? undefined,   // null -> omitted
  };
}
```

### 5 — Reset to page 1 on filter change

Always reset to page 1 when a filter changes, but keep the current pageSize and sort.

```typescript
private lastPagination = signal<PaginationParams | null>(null);

onPageChange(params: PaginationParams): void {
  this.lastPagination.set(params);
  this.load$.next(this.buildParams(params));
}

applyFilters(): void {
  const last = this.lastPagination();
  this.load$.next(this.buildParams({
    page: 1,
    pageSize:   last?.pageSize   ?? 15,
    sortColumn: last?.sortColumn,
    sortOrder:  last?.sortOrder,
  }));
}
```

### 6 — Clear filters

Reset every control to its initial value explicitly, then re-apply immediately.
Always pass the full reset object — never rely on `reset()` without defaults.

```typescript
clearFilters(): void {
  this.filters.reset({
    searchTerm:   '',
    categoryName: '',
    type:         null,  // selects reset to null, not ''
    status:       null,
  });
  this.applyFilters();   // fire immediately, bypasses the debounce
}
```

---

## Skeleton Reference — `p-skeleton`

```html
<!-- Single row bar -->
<p-skeleton width="100%" height="2.5rem" styleClass="mb-2" />

<!-- Short label / title -->
<p-skeleton width="40%" height="1.25rem" styleClass="mb-4" />

<!-- Typical table skeleton (6 rows) -->
@for (_ of [1,2,3,4,5,6]; track $index) {
  <p-skeleton width="100%" height="2.5rem" styleClass="mb-2" />
}
```

Import: `SkeletonModule` from `primeng/skeleton`

---

## Toast Reference — `ToastService`

Located at: `@core/helpers/toast.service`

```typescript
private toast = inject(ToastService);

this.toast.showSuccessToast('Success', 'Record saved successfully.');
this.toast.showErrorToast('Error', 'Something went wrong.');
this.toast.showWarningToast('Warning', 'This action cannot be undone.');
this.toast.showInfoToast('Info', 'Page refreshed.');

// Persistent error (does not auto-dismiss)
this.toast.showErrorToast('Error', 'Critical failure.', true);
```

---

## Rules

- ✅ Use `Subject<PaginationParams>` + `switchMap` — never nested subscribes for loading
- ✅ `debounceTime(800)` + `distinctUntilChanged` on `filters.valueChanges` for auto-apply
- ✅ `buildParams()` merges pagination + filter values — keep them separate until that point
- ✅ Use `|| undefined` for strings, `?? undefined` for numbers when building params
- ✅ Store `lastPagination` signal so filter changes always reset to page 1
- ✅ Dependent select: reset child with `{ emitEvent: false }` to avoid a double request
- ✅ `clearFilters()` passes full reset object — never call bare `reset()` without defaults
- ✅ Three exclusive states: `isLoading` -> `hasError` -> data
- ✅ `catchError` inside the `switchMap` pipe — not in subscribe callbacks
- ✅ `toast.showErrorToast` on every failure
- ✅ `toast.showSuccessToast` after every destructive action (delete, bulk update, etc.)
- ✅ Inline retry button on error state
- ✅ Use `p-card` — never `mat-card`
- ✅ `takeUntilDestroyed(this.destroyRef)` on every subscription
- ✅ Every interactive element has a `data-cy` attribute — format: `[feature]-[field]-[element-type]`
- ✅ PrimeNG selects in filter bar use `[inputAttrs]="{ 'data-cy': '...' }"`
- ❌ Never use `p-progress-spinner` as the only loading state — use skeleton rows
- ❌ Never swallow errors silently — always call `toast.showErrorToast`
- ❌ Never call `applyFilters()` directly inside `listenForFilterChanges()` subscribe — let debounce handle the timing
