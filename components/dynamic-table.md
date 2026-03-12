# Dynamic Table Component

**Selector:** `eretail-dynamic-table`  
**File:** `src/app/shared/components/dynamic-table/dynamic-table.ts`  
**Library:** Angular Material Table + Paginator + Sort

The `DynamicTableComponent` is the **primary data table** for listing pages.
It handles: columns, pagination, filtering, row selection, and action buttons.

---

## Inputs

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `tableData` | `any[]` | ✅ | — | Array of data rows |
| `tableColumns` | `EretailTableColumn[]` | ✅ | — | Column definitions |
| `tableTitle` | `string` | No | `''` | Title shown in table header |
| `tableSubTitle` | `string` | No | `''` | Subtitle in header |
| `tableButtons` | `TableButton[]` | No | `[]` | Action buttons per row |
| `filterFields` | `TableFilterField[]` | No | `[]` | Filter fields above table |
| `displaySearchBar` | `boolean` | No | `true` | Show/hide the search bar area |
| `emptyDataMsg` | `string` | No | `''` | Message when no data |
| `isSelectItems` | `boolean` | No | `false` | Enable row checkboxes |
| `isGetByPagination` | `boolean` | No | — | Use server-side pagination |
| `columnTemplates` | `{ [key: string]: TemplateRef }` | No | `{}` | Custom cell templates |

## Outputs

| Output | Type | Description |
|---|---|---|
| `pageChanged` | `{ currentPage, pageSize }` | Fired on pagination change |
| `onFiltering` | `FormGroup` | Fired when filter is applied |

---

## Type Definitions

### `EretailTableColumn`

```typescript
interface EretailTableColumn {
  label: string;          // Column header text
  property: {
    value: any;           // Key in data object (supports dot notation for nested)
    type: string | null;  // Cell type: null | 'anchor' | 'chips' | 'defaultWithIcon'
  };
}
```

### `TableButton<T>`

```typescript
interface TableButton<T = any> {
  icon: string;                      // Bootstrap icon name (e.g., 'pencil')
  tooltip?: string;                  // Tooltip text
  action: (data: T) => void;         // Callback on click
  hidden?: (data: T) => boolean;     // Conditionally hide button
  color?: string;                    // Icon color
}
```

### `TableFilterField`

```typescript
interface TableFilterField {
  id: string;
  label?: string;
  placeholder?: string;
  fieldType?: 'text' | 'select' | 'date' | 'number' | 'email' | 'password';
  formControl: FormControl;
  selectOptions?: Array<{ value: any; label: string }>;
  hidden?: boolean;
  classList?: string;
  fill?: boolean;
  icon?: string;
}
```

---

## Usage Examples

### Basic Table (Read-only, no actions)

```typescript
@Component({
  selector: 'app-product-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <content-wrapper>
      <eretail-dynamic-table
        [tableData]="products()"
        [tableColumns]="columns"
        tableTitle="Products"
      />
    </content-wrapper>
  `,
  imports: [DynamicTableComponent, ContentWrapperComponent],
})
export class ProductListComponent {
  products = signal<Product[]>([]);

  columns: EretailTableColumn[] = [
    { label: 'Name', property: { value: 'name', type: null } },
    { label: 'SKU', property: { value: 'sku', type: null } },
    { label: 'Status', property: { value: 'status', type: 'chips' } },
  ];
}
```

### Table with Action Buttons

```typescript
columns: EretailTableColumn[] = [
  { label: 'Name', property: { value: 'name', type: null } },
  { label: 'Price', property: { value: 'price', type: null } },
];

buttons: TableButton<Product>[] = [
  {
    icon: 'pencil',
    tooltip: 'Edit',
    action: (row) => this.router.navigate(['/products/edit', row.id]),
  },
  {
    icon: 'trash',
    tooltip: 'Delete',
    action: (row) => this.onDelete(row),
    hidden: (row) => !row.canDelete,
    color: 'red',
  },
];
```

```html
<eretail-dynamic-table
  [tableData]="products()"
  [tableColumns]="columns"
  [tableButtons]="buttons"
  tableTitle="Products"
/>
```

### Table with Filters

```typescript
filterFields: TableFilterField[] = [
  {
    id: 'name',
    label: 'Name',
    placeholder: 'Search by name...',
    fieldType: 'text',
    formControl: new FormControl(''),
  },
  {
    id: 'status',
    label: 'Status',
    fieldType: 'select',
    formControl: new FormControl(null),
    selectOptions: [
      { value: 'active', label: 'Active' },
      { value: 'inactive', label: 'Inactive' },
    ],
  },
];
```

```html
<eretail-dynamic-table
  [tableData]="products()"
  [tableColumns]="columns"
  [filterFields]="filterFields"
  (onFiltering)="onFilter($event)"
/>
```

### Cell Types

```typescript
// Chips cell: value must be { value: string, chipStyle: 'default' | 'success' | 'danger' | 'warn' }
{ label: 'Status', property: { value: 'status', type: 'chips' } }

// Anchor cell: value must be a string (the display text), url is hardcoded
{ label: 'Name', property: { value: 'name', type: 'anchor' } }

// Icon + Text cell: value must be { value: string, iconClass: string, iconText: string }
{ label: 'Type', property: { value: 'type', type: 'defaultWithIcon' } }
```

---

## Rules

- ✅ Use `eretail-dynamic-table` as the standard table for all listing pages
- ✅ Always use `signal<T[]>([])` for `tableData` source
- ✅ Define `columns` and `buttons` as class properties (not inside template)
- ✅ Use `(onFiltering)` + `(pageChanged)` for server-side data loading
- ❌ Never use `mat-table` directly in feature pages — use this wrapper
- ❌ Never embed business logic inside `TableButton.action` — call a method
