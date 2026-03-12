# Pattern: Listing Page

The standard pattern for pages that list/query data from an API.
Uses skeleton loading, inline error state, and `ToastService` for operation feedback.

## Structure

```
feature-list/
├── feature-list.component.ts    # Main component
├── feature-list.component.html  # Template
└── feature-list.component.scss  # Styles (usually empty for list pages)
```

---

## TypeScript Template

```typescript
import {
  Component, OnInit, inject, signal,
  ChangeDetectionStrategy, DestroyRef
} from '@angular/core';
import { Router } from '@angular/router';
import { FormControl } from '@angular/forms';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { catchError, finalize, of } from 'rxjs';
import { SkeletonModule } from 'primeng/skeleton';
import { CardModule } from 'primeng/card';
import { MessageModule } from 'primeng/message';
import { ContentWrapperComponent } from '@shared/components/content-wrapper/content-wrapper.component';
import { HeadingComponent } from '@shared/components/heading/heading.component';
import { CustomButtonComponent } from '@shared/components/button/button/button.component';
import {
  DynamicTableComponent,
  EretailTableColumn,
  TableButton,
  TableFilterField,
} from '@shared/components/dynamic-table/dynamic-table';
import { ToastService } from '@core/helpers/toast.service';
import { FeatureService } from '@core/services/feature.service';
import { Feature } from '@core/models/feature.model';

@Component({
  selector: 'app-feature-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    ContentWrapperComponent,
    HeadingComponent,
    CustomButtonComponent,
    DynamicTableComponent,
    SkeletonModule,
    CardModule,
    MessageModule,
  ],
  templateUrl: './feature-list.component.html',
})
export class FeatureListComponent implements OnInit {
  private service     = inject(FeatureService);
  private router      = inject(Router);
  private toast       = inject(ToastService);
  private destroyRef  = inject(DestroyRef);

  items      = signal<Feature[]>([]);
  isLoading  = signal(true);
  hasError   = signal(false);

  // Skeleton rows: array used only for @for iteration
  readonly skeletonRows = Array(6).fill(null);

  columns: EretailTableColumn[] = [
    { label: 'Name',   property: { value: 'name',      type: null } },
    { label: 'Status', property: { value: 'status',    type: 'chips' } },
    { label: 'Date',   property: { value: 'createdAt', type: null } },
  ];

  buttons: TableButton<Feature>[] = [
    {
      icon: 'pencil',
      tooltip: 'Edit',
      action: (row) => this.router.navigate(['/feature/edit', row.id]),
    },
    {
      icon: 'trash',
      tooltip: 'Delete',
      action: (row) => this.onDelete(row),
      color: 'red',
    },
  ];

  filterFields: TableFilterField[] = [
    {
      id: 'name',
      label: 'Name',
      placeholder: 'Search...',
      fieldType: 'text',
      formControl: new FormControl(''),
    },
  ];

  ngOnInit(): void {
    this.loadData();
  }

  private loadData(): void {
    this.isLoading.set(true);
    this.hasError.set(false);

    this.service.getAll().pipe(
      catchError(() => {
        this.hasError.set(true);
        this.toast.showErrorToast('Error', 'Failed to load features. Please try again.');
        return of([]);
      }),
      finalize(() => this.isLoading.set(false)),
      takeUntilDestroyed(this.destroyRef),
    ).subscribe(data => this.items.set(data));
  }

  onFilter(_filterForm: any): void {
    this.loadData();
  }

  onRetry(): void {
    this.loadData();
  }

  private onDelete(item: Feature): void {
    this.service.delete(item.id).pipe(
      takeUntilDestroyed(this.destroyRef),
    ).subscribe({
      next: () => {
        this.items.update(prev => prev.filter(i => i.id !== item.id));
        this.toast.showSuccessToast('Success', 'Feature deleted successfully.');
      },
      error: () => {
        this.toast.showErrorToast('Error', 'Failed to delete the feature.');
      },
    });
  }
}
```

---

## HTML Template

```html
<content-wrapper>
  <div class="d-flex justify-content-between align-items-center mb-3">
    <app-heading title="Feature List" subtitle="Manage all features" />

    <app-button
      buttonClass="bt-primary"
      icon="plus"
      routerLink="/feature/create">
      New Feature
    </app-button>
  </div>

  <!-- ── Loading state: skeleton ── -->
  @if (isLoading()) {
    <p-card>
      <p-skeleton width="40%" height="1.25rem" styleClass="mb-4" />
      @for (_ of skeletonRows; track $index) {
        <p-skeleton width="100%" height="2.5rem" styleClass="mb-2" />
      }
    </p-card>
  }

  <!-- ── Error state: inline message with retry ── -->
  @else if (hasError()) {
    <p-card>
      <div class="d-flex flex-column align-items-center gap-3 py-4">
        <p-message
          severity="error"
          text="Failed to load the data. Check your connection and try again." />
        <app-button buttonClass="bt-default" icon="arrow-clockwise" (clicked)="onRetry()">
          Try again
        </app-button>
      </div>
    </p-card>
  }

  <!-- ── Success state: data table ── -->
  @else {
    <p-card>
      <eretail-dynamic-table
        [tableData]="items()"
        [tableColumns]="columns"
        [tableButtons]="buttons"
        [filterFields]="filterFields"
        tableTitle="Features"
        (onFiltering)="onFilter($event)"
      />
    </p-card>
  }
</content-wrapper>
```

---

## Skeleton Reference — `p-skeleton`

```html
<!-- Single row bar -->
<p-skeleton width="100%" height="2.5rem" styleClass="mb-2" />

<!-- Short label -->
<p-skeleton width="40%" height="1.25rem" styleClass="mb-4" />

<!-- Circle (avatar) -->
<p-skeleton shape="circle" size="3rem" styleClass="me-2" />

<!-- Card block -->
<p-skeleton width="100%" height="280px" styleClass="mb-3" />

<!-- Typical table skeleton (6 rows) -->
@for (_ of skeletonRows; track $index) {
  <p-skeleton width="100%" height="2.5rem" styleClass="mb-2" />
}
```

Import: `SkeletonModule` from `primeng/skeleton`

---

## Toast Reference — `ToastService`

Located at: `@core/helpers/toast.service`

```typescript
private toast = inject(ToastService);

// Usage
this.toast.showSuccessToast('Success', 'Record saved successfully.');
this.toast.showErrorToast('Error', 'Something went wrong.');
this.toast.showWarningToast('Warning', 'This action cannot be undone.');
this.toast.showInfoToast('Info', 'Page refreshed.');

// Persistent error (does not auto-dismiss)
this.toast.showErrorToast('Error', 'Critical failure.', true);
```

---

## Rules

- ✅ Start with `isLoading = signal(true)` — skeleton shows immediately on first render
- ✅ Use `skeletonRows = Array(N).fill(null)` for `@for` iteration — N matches expected row count
- ✅ Three exclusive states: `isLoading` → `hasError` → data
- ✅ Use `catchError` + `finalize` in the pipe — never in subscribe callbacks
- ✅ Use `toastService.showErrorToast` for all error feedback
- ✅ Use `toastService.showSuccessToast` after every destructive action (delete, save, etc.)
- ✅ Show inline retry button on page error state
- ✅ Import `SkeletonModule` from `primeng/skeleton`
- ✅ Use `p-card` — never `mat-card`
- ✅ Use `takeUntilDestroyed(this.destroyRef)` on every subscription
- ❌ Never use `p-progress-spinner` as the only loading state for a list page — use skeleton
- ❌ Never swallow errors silently — always call `toast.showErrorToast`
