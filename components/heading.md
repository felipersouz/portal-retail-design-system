# Heading Component

**Selector:** `app-heading`  
**File:** `src/app/shared/components/heading/heading.component.ts`

Used for **page-level titles**. Always use this instead of raw `<h1>` + `<p>` combinations.

---

## Inputs

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `title` | `string` | ✅ | — | Main page title |
| `subtitle` | `string` | No | — | Subtitle / description |
| `size` | `'small' \| 'medium' \| 'large'` | No | `'medium'` | Controls font scale |

---

## Usage

```html
<!-- Standard page header -->
<app-heading
  title="Product List"
  subtitle="Manage all products" />

<!-- Without subtitle -->
<app-heading title="Settings" />

<!-- Small variant (for section headings inside a page) -->
<app-heading title="General Info" size="small" />
```

---

## Rules

- ✅ Use `app-heading` at the top of every page component
- ✅ Keep `title` short and descriptive (max ~40 chars)
- ✅ Use `subtitle` to provide context or instructions
- ❌ Never use raw `<h1 class="text-title">` in page templates
