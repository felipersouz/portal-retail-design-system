# Prompt: Create Reusable Component

Use this prompt when you need to build a shared/reusable component that will be consumed
across multiple pages and features. The goal is maximum abstraction, strong typing, and
extensibility without modifying component internals.

The reference architecture in this project is `app-table` (container) + `app-table-cell` (renderer).

---

## Before starting

Read these design system files from `felipersouz/portal-retail-design-system`:
- `guidelines/reusable-components.md` — architecture patterns (required)
- `guidelines/angular-architecture.md` — Angular 20 rules
- `guidelines/angular-modern-features.md` — signals, viewChild, model
- `guidelines/code-quality.md` — typing and naming

---

## Prompt template

```
Create a reusable Angular component for [COMPONENT PURPOSE].

Read `guidelines/reusable-components.md` from felipersouz/portal-retail-design-system
before writing any code.

--- COMPONENT SPEC ---

Name: [component-name]
Selector: app-[component-name]
Location: src/app/shared/components/[component-name]/

Purpose: [describe what it renders and in what contexts it will be used]

Data shape:
[describe the data model the component will receive, e.g.:
  - title: string
  - value: number
  - status: 'active' | 'inactive'
  - trend?: number (percentage)
]

Variations / types needed:
[list all the rendering variations, e.g.:
  - type: 'text' | 'currency' | 'date' | 'badge' | 'boolean'
  - size: 'sm' | 'md' | 'lg'
  - severity: 'success' | 'danger' | 'info' | 'warn'
]

Consumer will need to:
[describe what the consumer configures from outside, e.g.:
  - Pass a list of items with a config object per item
  - Control which columns/fields are visible
  - Provide custom cell rendering for specific fields
  - React to click/select events
]

Loading state: [yes / no — skeleton built into the component]
Pagination: [yes / no]
Sorting: [yes / no]
Selection: [yes / no — single / multiple]

--- ARCHITECTURE REQUIREMENTS ---

Follow the reusable-components.md guidelines exactly:

1. INTERFACES FIRST
   - Define and export all configuration types and interfaces at the top of the file
   - Use union types for all variation axes (type, size, severity, etc.)
   - Use generic <T> if the component works over a list of items
   - Use `field: keyof T & string` for field references — not plain string

2. COMPOSITION
   - Create a container component for structure, state, and loading
   - Create a sub-component for each independently renderable unit (e.g. cell, card, item)
   - Sub-component receives only the data it needs — no parent references

3. INPUTS AND OUTPUTS
   - Use input() and output() — never @Input() / @Output()
   - Every input() must have a default value where appropriate
   - Outputs use typed event interfaces — not raw primitives
   - input.required<T>() for truly required data

4. SKELETON LOADING
   - isLoading = input(false) on the container
   - skeletonItems = computed(() => Array.from({ length: N }, (_, i) => i))
   - Skeleton row count derived from a size or pageSize input — never hardcoded
   - Skeleton renders in place of real content — same layout, matching heights

5. ESCAPE HATCH
   - Add at least one way to customize rendering without modifying the component:
     Option A: custom type in the ColumnType union + consumer provides ng-template
     Option B: contentChild<TemplateRef<any>>() for named template slots
     Option C: renderFn?: (value: T) => string for simple transformations

6. ANGULAR RULES
   - ChangeDetectionStrategy.OnPush on every component
   - inject() — never constructor injection
   - signal() / computed() for all state
   - @if / @for with track — never *ngIf / *ngFor
   - PrimeNG for all UI primitives (p-tag, p-skeleton, p-card, p-button, p-badge)
   - No Angular Material

--- OUTPUT FORMAT ---

Produce the following files:
1. src/app/shared/components/[name]/[name].ts — interfaces + container component
2. src/app/shared/components/[name]/[name].html — container template
3. src/app/shared/components/[name]/[name].scss — styles (minimal)
4. src/app/shared/components/[name]/[name]-cell/[name]-cell.ts — sub-component (if needed)
5. src/app/shared/components/[name]/[name]-cell/[name]-cell.html — sub-component template

After the files, provide:
- Usage example showing how a consumer would use the component in a page
- List of exported interfaces the consumer needs to import
- How to add a new variation in the future (e.g. new column type)
```

---

## Concrete example: Status Badge List

```
Create a reusable Angular component for displaying a list of items with configurable
status badges, icons, and values.

Read `guidelines/reusable-components.md` from felipersouz/portal-retail-design-system
before writing any code.

Name: item-status-list
Selector: app-item-status-list
Location: src/app/shared/components/item-status-list/

Purpose: Displays a vertical or horizontal list of labeled items, each with a configurable
badge/tag, icon, and optional trend indicator. Used in summary panels, sidebars, and dashboards.

Data shape:
  - label: string
  - value: string | number
  - status?: string         (maps to a badge severity)
  - icon?: string           (Bootstrap icon class)
  - trend?: number          (positive = green arrow, negative = red arrow)

Variations needed:
  - layout: 'vertical' | 'horizontal'
  - size: 'sm' | 'md'
  - badgeSeverityMap: Record<string, BadgeSeverity>  (consumer-defined mapping)

Consumer configures:
  - items: ItemStatusConfig[]
  - layout and size
  - badgeSeverityMap (e.g. { active: 'success', inactive: 'danger' })

Loading state: yes — skeleton with N rows matching items.length
Architecture requirements: follow guidelines/reusable-components.md exactly.
```

---

## Tips

- **Always read `guidelines/reusable-components.md` first** — the AI uses it to structure the component correctly
- For table-like components, study `app-table` + `app-table-cell` as a reference
- Prefer `input.required<T>()` for the main data input — `input<T[]>([])` for optional lists
- When in doubt about escape hatches, add both: a `custom` type AND a `contentChild` template slot
- If you add a new `ColumnType` later, only `app-table-cell` needs to change — nothing else
