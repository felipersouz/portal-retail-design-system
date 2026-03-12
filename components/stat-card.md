# Stat Card Component

**Selector:** `app-stat-card`  
**File:** `src/app/shared/components/stat-card/stat-card.ts`

Displays a group of labeled KPI/metric cards on dashboard and summary pages.

---

## Interfaces

```typescript
interface StatCard {
  label: string;
  value: number;
  color: 'yellow' | 'red' | 'green' | 'blue' | 'purple' | 'orange';
}
```

## Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `title` | `string` | ✅ | Section title above the cards |
| `stats` | `StatCard[]` | ✅ | Array of stat items to display |

---

## Usage

```typescript
@Component({
  selector: 'app-dashboard',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <content-wrapper>
      <app-stat-card
        title="Sales Overview"
        [stats]="salesStats()" />
    </content-wrapper>
  `,
  imports: [StatCardComponent, ContentWrapperComponent],
})
export class DashboardComponent {
  salesStats = signal<StatCard[]>([
    { label: 'Total Orders', value: 1280, color: 'blue' },
    { label: 'Pending',      value: 43,   color: 'yellow' },
    { label: 'Delivered',    value: 1100, color: 'green' },
    { label: 'Cancelled',    value: 137,  color: 'red' },
  ]);
}
```

---

## Rules

- ✅ Use `app-stat-card` on all dashboard and summary pages
- ✅ Always use signals for `stats` data source
- ✅ Choose `color` semantically: `green` = positive, `red` = negative, `yellow` = warning
