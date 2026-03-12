# Spacing & Layout Tokens

Spacing in this project follows Bootstrap 5's utility system plus project-specific patterns.

---

## Spacing System

Use **Bootstrap 5 spacing utilities** (`m-`, `p-`, `gap-`) as the primary spacing mechanism.

| Class | Value |
|---|---|
| `m-0` / `p-0` | 0 |
| `m-1` / `p-1` | 0.25rem (4px) |
| `m-2` / `p-2` | 0.5rem (8px) |
| `m-3` / `p-3` | 1rem (16px) |
| `m-4` / `p-4` | 1.5rem (24px) |
| `m-5` / `p-5` | 3rem (48px) |

---

## Project-Specific Layout Patterns

### Content Wrapper

All page content must be wrapped in `<content-wrapper>` for consistent padding.

```html
<content-wrapper>
  <!-- page content here -->
</content-wrapper>
```

The `basePadding` input (default: `true`) applies standard inner padding.

```typescript
// Disable default padding when you need full-width content
<content-wrapper [basePadding]="false">
```

### Form Layout

```scss
.eretail-form-sm {
  max-width: 1100px;
  width: 100%;
  padding-inline: 1rem;
}
```

Use `.eretail-form-sm` on `<form>` elements to constrain width on wide screens.

### Card Layout

```scss
.card-header-custom {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 1.5rem;
}
```

### Table Header Layout

```scss
.header-table {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  min-height: 64px;
}
```

### Common Box Shadow

```scss
.box-shadow { box-shadow: rgba(99, 99, 99, 0.2) 0px 2px 8px 0px; }
```

---

## Angular Material Customizations

### mat-card

```scss
border-radius: 1.1rem;  // Project override
```

### mat-tab-body-content

```scss
padding: 2.5rem 1rem 2rem 2rem;  // Tab content area
gap: 2rem;  // Between tab children
```

### Chips

```scss
border-radius: 0.3125rem;   // Rounded-sm (not fully rounded)
min-height: 1.75rem;
max-height: 2.25rem;
```

### Table Cells

```scss
padding: 8px;  // mat-table cells override
```

---

## Rules

- ✅ Use Bootstrap utility classes for margins and paddings
- ✅ Always wrap page content in `<content-wrapper>`
- ✅ Use `gap` on flex/grid containers instead of margins between children
- ❌ Never use hardcoded `px` values for spacing in component SCSS — use `rem`
- ❌ Never apply padding directly to page root element — use `content-wrapper`
