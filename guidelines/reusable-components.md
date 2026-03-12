# Reusable Component Guidelines

Principles for building generic, scalable, and reusable Angular components in portal-retail.
The reference implementation is `app-table` + `app-table-cell`.

---

## Core Principles

### 1. Configuration over code

The consumer should never touch the component internals to change its behavior.
All variation is expressed via typed configuration objects passed as `input()`.

```typescript
// ❌ Consumer forks the component to add a new column type
// ✅ Consumer passes a config object — component handles it generically
columns: ColumnConfig<Product>[] = [
  { field: 'name',   header: 'Name',   type: 'text' },
  { field: 'price',  header: 'Price',  type: 'currency' },
  { field: 'status', header: 'Status', type: 'badge', badgeMap: { active: 'success' } },
];
```

### 2. Composition over inheritance

Split into a **container component** (structure, state, pagination) and
**cell/item components** (rendering a single unit of data).

```
app-table/
├── table.ts             ← container: structure, pagination, skeleton, actions
├── table.html
├── table.scss
└── table-cell/
    ├── table-cell.ts    ← renderer: one cell, all types
    ├── table-cell.html
    └── table-cell.scss
```

### 3. Interfaces are the public API

Define all configuration interfaces **exported from the component file**,
so consumers get full type safety without importing internals.

```typescript
// ✅ Exported — consumers can import and use them
export type ColumnType = 'text' | 'date' | 'currency' | 'boolean' | 'badge' | 'custom';
export type BadgeSeverity = 'success' | 'danger' | 'info' | 'warn' | 'secondary';

export interface ColumnConfig<T = any> {
  field: keyof T & string;            // strongly typed to the data model
  header: string;
  type?: ColumnType;
  sortable?: boolean;
  badgeMap?: Record<string, BadgeSeverity>;
  enumDisplayMap?: Record<string | number, string>;
  width?: string;
}

export interface ActionConfig<T = any> {
  icon: string;
  label?: string;
  severity?: ButtonSeverity;
  visible?: (row: T) => boolean;      // consumer controls visibility per row
  disabled?: (row: T) => boolean;
  action: (row: T) => void;
}
```

### 4. Generic typing `<T>` for the data shape

When the component works over a list of items, use a generic type parameter
so TypeScript can enforce `field` values and `action` row types.

```typescript
// ✅ Generic — compiler catches wrong field names at build time
export interface ColumnConfig<T = any> {
  field: keyof T & string;
}

// Consumer usage:
columns: ColumnConfig<Product>[] = [
  { field: 'name' },     // ✅ valid
  { field: 'xyzz' },     // ❌ TypeScript error — not a key of Product
];
```

### 5. Smart defaults + escape hatches

Every `input()` should have a sensible default.
Add an escape hatch (`ng-template` / `custom` type) for cases the component does not cover.

```typescript
// Smart defaults
isLoading  = input(false);
pageSize   = input(10);
sortable   = input(true);
hasPagination = input(true);

// Escape hatch: consumer provides a custom template for a specific column
// <app-table>
//   <ng-template appCellTemplate="actions" let-row>
//     <my-custom-button [row]="row" />
//   </ng-template>
// </app-table>
```

### 6. Skeleton built into the component

The component owns its loading state — the consumer only passes `isLoading`.
Skeleton rows/items are generated via `computed()`.

```typescript
// ✅ Skeleton rows derived from current page size — no consumer involvement
skeletonRows = computed(() =>
  Array.from({ length: this.currentPageSize() }, (_, i) => i)
);
```

```html
<!-- Table body renders skeleton OR real data — consumer never writes loading logic -->
@if (isLoading()) {
  <p-skeleton width="100%" height="1rem" />
} @else {
  <app-table-cell [value]="row[col.field]" [type]="col.type" />
}
```

### 7. Outputs are typed events, not raw values

```typescript
// ✅ Typed output — consumer knows exactly what they receive
pageChange = output<PaginationEvent>();
rowClick   = output<T>();
selected   = output<T[]>();

export interface PaginationEvent {
  page: number;
  pageSize: number;
  sortColumn?: string;
  sortOrder?: 'asc' | 'desc';
}
```

---

## Reference Implementation: `app-table` + `app-table-cell`

### What makes it scalable

| Feature | How it is implemented |
|---|---|
| New column type (badge, enum, date…) | Add to `ColumnType` union + handle in `TableCell` |
| New action style per row | Pass `visible: (row) => boolean` in `ActionConfig` |
| Custom cell rendering | `columnTemplates` input — consumer passes `TemplateRef` |
| Badge color per value | `badgeMap: Record<string, BadgeSeverity>` in column config |
| Enum label display | `enumDisplayMap: Record<string|number, string>` |
| Sort behavior per column | `is_sortable?: boolean` in `TableColumns` |
| Pagination control | `hasPagination`, `rowPerPageOptions`, `pageSize` inputs |
| Skeleton rows count | Derived from `currentPageSize()` via `computed()` |

### Anatomy

```
Consumer → passes ColumnConfig[], ActionConfig[], data T[], isLoading
    ↓
app-table (container)
  - holds pagination state (currentPage, currentPageSize — signals)
  - computes displayedColumns, skeletonRows
  - emits pageChange, rowClick
  - renders <p-table> structure
    ↓
  app-table-cell (renderer)
    - receives value, type, badgeMap, enumDisplayMap
    - renders the correct UI for each type
    - @switch on type → text / date / currency / badge / enum / boolean
```

---

## Step-by-step: Building a new reusable component

### Step 1 — Define the public interface first

Before writing any template or class logic, define all exported types and interfaces.
Ask yourself: "What is the minimum a consumer needs to pass to make this useful?"

```typescript
// component-name.ts — define interfaces at the top, exported

export type StatusType = 'active' | 'inactive' | 'pending';
export type CardSize = 'sm' | 'md' | 'lg';

export interface StatCardConfig {
  title: string;
  value: number | string;
  icon?: string;
  trend?: number;          // percentage change
  trendLabel?: string;
  size?: CardSize;         // default: 'md'
}
```

### Step 2 — Identify sub-components

Ask: "Is there a unit inside this component that renders independently?"
If yes, extract it as a child component.

```
app-stat-card-grid/        ← container: grid layout, loading state, empty state
├── stat-card-grid.ts
└── stat-card/             ← unit: renders one card
    └── stat-card.ts
```

### Step 3 — Design inputs and outputs

```typescript
// Container
@Component({ selector: 'app-stat-card-grid', ... })
export class StatCardGridComponent {
  cards     = input.required<StatCardConfig[]>();
  isLoading = input(false);
  columns   = input<2 | 3 | 4>(3);          // grid columns
  cardClick = output<StatCardConfig>();
}

// Unit
@Component({ selector: 'app-stat-card', ... })
export class StatCardComponent {
  config  = input.required<StatCardConfig>();
  size    = input<CardSize>('md');
  clicked = output<StatCardConfig>();
}
```

### Step 4 — Add skeleton at the container level

```typescript
skeletonCards = computed(() =>
  Array.from({ length: this.columns() * 2 }, (_, i) => i)
);
```

```html
@if (isLoading()) {
  @for (_ of skeletonCards(); track $index) {
    <p-skeleton height="7rem" styleClass="mb-2" />
  }
} @else {
  @for (card of cards(); track card.title) {
    <app-stat-card [config]="card" (clicked)="cardClick.emit($event)" />
  }
}
```

### Step 5 — Add the escape hatch

For cells or slots a consumer may need to customize, use `ng-template` + `@ContentChild`:

```typescript
// In the container
customHeader = contentChild<TemplateRef<any>>('headerTemplate');
```

```html
<!-- Container template -->
@if (customHeader()) {
  <ng-container [ngTemplateOutlet]="customHeader()!" />
} @else {
  <h3>{{ title() }}</h3>
}
```

```html
<!-- Consumer usage -->
<app-stat-card-grid [cards]="cards">
  <ng-template #headerTemplate>
    <div class="my-custom-header">...</div>
  </ng-template>
</app-stat-card-grid>
```

---

## Checklist — before submitting a reusable component

- [ ] All configuration interfaces are exported from the component file
- [ ] Generic `<T>` used when the component works over a list of items
- [ ] `field` property uses `keyof T & string` for type safety
- [ ] All inputs have default values where appropriate
- [ ] Skeleton is built into the component — consumer only passes `isLoading`
- [ ] `skeletonRows/skeletonItems` computed from a size input — not hardcoded
- [ ] Outputs use typed interfaces, not raw primitives
- [ ] At least one escape hatch exists (`ng-template` / `custom` type) for extensibility
- [ ] Sub-components extracted for independently renderable units
- [ ] `ChangeDetectionStrategy.OnPush` on every component and sub-component
- [ ] No Angular Material — use PrimeNG for all UI primitives
- [ ] `input()` / `output()` — never `@Input()` / `@Output()`
- [ ] `inject()` — never constructor injection
