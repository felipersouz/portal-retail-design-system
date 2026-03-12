# Typography Tokens

All typography tokens are defined in `src/app/styles/_variables.scss` and consumed via `_typography.scss`.

---

## Font Size Scale

| SCSS Variable | Value | Equivalent |
|---|---|---|
| `$font-size-h1` | `2rem` | 32px |
| `$font-size-h2` | `1.8rem` | 28.8px |
| `$font-size-h3` | `1.5rem` | 24px |
| `$font-size-h4` | `1.2rem` | 19.2px |
| `$font-size-p14` | `0.875rem` | 14px (body default) |
| `$font-size-p13` | `0.8125rem` | 13px |
| `$font-size-p12` | `0.75rem` | 12px (small labels) |

---

## Font Weight Scale

| SCSS Variable | Value | Usage |
|---|---|---|
| `$font-weight-bold` | `700` | Titles, strong emphasis |
| `$font-weight-semi-bold` | `600` | Section labels, card titles |
| `$font-weight-regular` | `400` | Body text, descriptions |

---

## Typography Utility Classes

### Size Classes

```scss
.size-h1     // 2rem, bold (700)
.size-h2     // 1.8rem, semi-bold (600)
.size-h3     // 1.5rem, bold (700)
.size-h4     // 1.2rem, regular (400)
.size-p13    // 0.8125rem, regular
.size-p12    // 0.75rem, regular
.size-p10    // 14px (legacy alias)
.text-normal // 14px
```

### Semantic Text Classes

```scss
.text-title              // 2rem, bold, color: #2a3680, margin: 1rem 0
.text-subtitle           // 0.875rem, regular, margin-bottom: 1rem
.text-form-section__label // 1.8rem, bold, color: #2a3680
.text-input__label        // 1.5rem, regular, color: $primary-default, margin-bottom: 0.35rem
```

---

## Usage Guidelines

### Page Titles

```html
<!-- Use the AppHeadingComponent for page titles -->
<app-heading title="Page Title" subtitle="Optional subtitle" />

<!-- Or use semantic class directly -->
<h1 class="text-title">Page Title</h1>
<p class="text-subtitle">Page subtitle or description</p>
```

### Form Section Labels

```html
<label class="text-form-section__label">Section Name</label>
```

### Input Labels

```html
<label class="text-input__label">Field Label</label>
```

### Body Text

```html
<p class="text-normal">Regular body content at 14px</p>
<span class="size-p12">Small labels or badges</span>
```

---

## Rules

- ✅ Use `.text-title` and `.text-subtitle` for page headers
- ✅ Use `.text-input__label` for labels above form fields
- ✅ Use `.text-form-section__label` to separate form sections
- ✅ Prefer `app-heading` component for page-level headings
- ❌ Never use `font-size` inline in component templates
- ❌ Never create new font-size values — extend `_variables.scss` instead
