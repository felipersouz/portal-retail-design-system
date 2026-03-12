# Content Wrapper Component

**Selector:** `content-wrapper`  
**File:** `src/app/shared/components/content-wrapper/content-wrapper.component.ts`

The standard page layout container. **All page content must be wrapped inside it.**
Provides consistent padding, max-width behavior, and layout structure.

---

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `basePadding` | `boolean` | `true` | Applies default inner padding when `true` |

---

## Usage

```html
<!-- Standard (default padding) -->
<content-wrapper>
  <app-heading title="Page Title" />
  
  <mat-card>
    <!-- content -->
  </mat-card>
</content-wrapper>

<!-- Full-width content (e.g., maps, full-bleed tables) -->
<content-wrapper [basePadding]="false">
  <leaflet-map />
</content-wrapper>
```

---

## Rules

- ✅ Every page component template must start with `<content-wrapper>`
- ✅ Place `app-heading` as the first child inside `content-wrapper`
- ❌ Never add `padding` or `margin` to the page root element directly
