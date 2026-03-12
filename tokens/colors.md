# Color Tokens

This document defines all color tokens used in the portal-retail-frontend project.

## How Colors Work

Colors are defined as **CSS Custom Properties** (`--variable`) set at runtime per client (white-label system).
SCSS variables map to these CSS variables. Never hardcode hex values in components — always use the SCSS variables or CSS variables below.

---

## Semantic Colors (Client-theming)

These colors are injected by the client configuration and vary per client.

| SCSS Variable | CSS Variable | Usage |
|---|---|---|
| `$primary` | `var(--primary)` | Main brand color (buttons, highlights) |
| `$border-primary` | `var(--border-primary)` | Borders that use brand color |
| `$primary2` | `var(--primary2)` | Secondary brand shade |
| `$secondary` | `var(--secondary)` | Secondary actions, accents |
| `$primary-white` | `var(--primary-white)` | Backgrounds, card surfaces |
| `$secondary-white` | `var(--secondary-white)` | Subtle backgrounds |
| `$primary-black` | `var(--primary-black)` | Primary text |
| `$primary-default` | `var(--primary-default)` | Default/neutral state |
| `$secondary-default` | `var(--secondary-default)` | Secondary neutral state |

### Feedback Colors

| SCSS Variable | CSS Variable | Usage |
|---|---|---|
| `$primary-danger` | `var(--primary-danger)` | Error states, delete actions |
| `$primary-danger2` | `var(--primary-danger2)` | Secondary error shade |
| `$secondary-danger` | `var(--secondary-danger)` | Error backgrounds |
| `$border-danger` | `var(--border-danger)` | Error borders |
| `$primary-warn` | `var(--primary-warn)` | Warning states |
| `$secondary-warn` | `var(--secondary-warn)` | Warning backgrounds |
| `$border-warn` | `var(--border-warn)` | Warning borders |
| `$primary-info` | `var(--primary-info)` | Informational states |
| `$secondary-info` | `var(--secondary-info)` | Info backgrounds |
| `$border-info` | `var(--border-info)` | Info borders |
| `$primary-success` | `var(--primary-success)` | Success states |
| `$secondary-success` | `var(--secondary-success)` | Success backgrounds |
| `$border-success` | `var(--border-success)` | Success borders |

### Utility Colors

| SCSS Variable | Value | Usage |
|---|---|---|
| `$alt-primary` | `#475bd2` | Fallback primary when CSS var not loaded |
| `$default-hover` | `rgba(208,208,208,0.15)` | Row/item hover state |
| `$gradient` | `var(--gradient)` | Gradient backgrounds |
| `$boxshadowtable` | `var(--boxshadowtable)` | Table box-shadow |

---

## Static Color Scale

These are global, fixed colors not tied to client theming.

### Black Scale (Text / Borders)

| CSS Variable | Hex | Usage |
|---|---|---|
| `--black-1` | `#000` | Deepest black |
| `--black-2` | `#111` | Very dark |
| `--black-3` | `#222` | Dark gray |
| `--black-4` | `#333` | Medium dark |
| `--black-5` | `#444` | Medium |
| `--black-6` | `#555` | Light medium |
| `--black-7` | `#666` | Light |
| `--black-8` | `#777` | Lighter |
| `--black-9` | `#888` | Very light |
| `--black-10` | `#999` | Near white gray |

### White Scale (Backgrounds)

| CSS Variable | Hex | Usage |
|---|---|---|
| `--white-1` | `#fff` | Pure white |
| `--white-2` | `#eee` | Near white |
| `--white-3` | `#ddd` | Light gray |
| `--white-4` | `#ccc` | Medium light |
| `--white-5` | `#bbb` | Medium |
| `--white-10` | `#666` | Near black |

---

## Utility CSS Classes

### Background Classes

```scss
.bg-primary-white   // $primary-white
.bg-primary         // $primary
.bg-secondary       // $secondary
.bg-default-light   // #e9e9e9 (static)
```

### Text Color Classes

```scss
.text-primary-defaut  // $primary (brand)
.text-primary         // $primary-black
.text-secondary       // $secondary
.text-value           // $secondary-default
.text-black           // $primary-black
.text-white           // $primary-white
.text-danger          // $primary-danger
.text-warn            // $primary-warn
.text-success         // $primary-success
```

### Black/White Scale Text Classes

```scss
.black-1 through .black-10  // CSS var color classes
.white-1 through .white-10  // CSS var color classes
```

---

## Rules

- ✅ Use SCSS variables (`$primary`, `$primary-danger`, etc.) in SCSS files
- ✅ Use CSS variables (`var(--primary)`) in inline styles when necessary
- ✅ Use utility classes (`.text-danger`, `.bg-primary`) in templates
- ❌ Never hardcode hex values in component `.scss` files
- ❌ Never set colors via `[style]` bindings in Angular templates when a CSS class exists
