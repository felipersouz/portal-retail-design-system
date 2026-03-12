# Pattern: Listing Page

The standard pattern for pages that list/query data from an API.

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
import { Component, OnInit, inject, signal, ChangeDetectionStrategy } from '@angular/core';
import { ChangeDetectorRef } from '@angular/core';
import { Router } from '@angular/router';
import { FormControl } from '@angular/forms';
import { ContentWrapperComponent } from '@shared/components/content-wrapper/content-wrapper.component';
import { HeadingComponent } from '@shared/components/heading/heading.component';
import { CustomButtonComponent } from '@shared/components/button/button/button.component';
import {
  DynamicTableComponent,
  EretailTableColumn,
  TableButton,
  TableFilterField,
} from '@shared/components/dynamic-table/dynamic-table';
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
  ],
  templateUrl: './feature-list.component.html',
})
export class FeatureListComponent implements OnInit {
  private service = inject(FeatureService);
  private router = inject(Router);

  items = signal<Feature[]>([]);
  isLoading = signal(false);

  columns: EretailTableColumn[] = [
    { label: 'Name',   property: { value: 'name',   type: null } },
    { label: 'Status', property: { value: 'status', type: 'chips' } },
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
    this.service.getAll().subscribe({
      next: (data) => {
        this.items.set(data);
        this.isLoading.set(false);
      },
      error: () => this.isLoading.set(false),
    });
  }

  onFilter(filterForm: any): void {
    // Apply filters and reload
    this.loadData();
  }

  private onDelete(item: Feature): void {
    this.service.delete(item.id).subscribe(() => {
      this.items.update(prev => prev.filter(i => i.id !== item.id));
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

  @if (isLoading()) {
    <div class="d-flex justify-content-center p-4">
      <p-progress-spinner />
    </div>
  } @else {
    <mat-card>
      <eretail-dynamic-table
        [tableData]="items()"
        [tableColumns]="columns"
        [tableButtons]="buttons"
        [filterFields]="filterFields"
        tableTitle="Features"
        (onFiltering)="onFilter($event)"
      />
    </mat-card>
  }
</content-wrapper>
```

---

## Rules

- ✅ Always show a loading state while fetching
- ✅ Define `columns`, `buttons`, and `filterFields` as class properties
- ✅ Use `signal<T[]>([])` for the data array
- ✅ Wrap in `<mat-card>` for the table container
- ❌ Never subscribe inside the template — use `async` pipe or `signal` + `subscribe`
