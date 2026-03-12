# File & Folder Structure

## Project Root (`src/app/`)

```
src/app/
├── core/                   # Singleton services, guards, interceptors, models
│   ├── config/             # Route configs for external users
│   ├── constants/          # App-wide constants
│   ├── directives/         # Core directives (not feature-specific)
│   ├── enums/              # All enums (one file per domain)
│   ├── functions/          # Pure utility functions
│   ├── guards/             # Route guards
│   ├── helpers/            # Helper classes/functions
│   ├── interceptors/       # HTTP interceptors
│   ├── models/             # TypeScript interfaces/types (one per domain)
│   ├── pipes/              # Global Angular pipes
│   ├── services/           # Feature services (API communication)
│   └── util/               # Misc utilities
│
├── pages/                  # Feature pages (lazy-loaded routes)
│   ├── feature-name/
│   │   ├── feature-name.routes.ts
│   │   ├── feature-list/
│   │   │   ├── feature-list.component.ts
│   │   │   ├── feature-list.component.html
│   │   │   └── feature-list.component.scss
│   │   └── feature-form/
│   │       ├── feature-form.component.ts
│   │       ├── feature-form.component.html
│   │       └── feature-form.component.scss
│
├── shared/                 # Reusable components and utilities
│   ├── components/         # Shared UI components
│   │   ├── button/
│   │   ├── dynamic-table/
│   │   ├── heading/
│   │   ├── stat-card/
│   │   └── content-wrapper/
│   ├── directives/         # Shared directives
│   ├── interfaces/         # Shared TypeScript interfaces
│   ├── modules/            # Legacy NgModules (tinymce, etc.)
│   ├── ui-components/      # Thin wrappers around library components
│   └── utils/              # Shared pure functions
│
└── styles/                 # Global SCSS
    ├── _variables.scss     # SCSS variables + font sizes
    ├── _colors.scss        # Color classes + CSS custom properties
    ├── _typography.scss    # Text/size utility classes
    ├── _buttons.scss       # Button styles (bt-primary, bt-danger, etc.)
    ├── _forms.scss         # Form layout styles
    ├── _card.scss          # Card header/layout styles
    ├── _table.scss         # Table layout helpers
    ├── _eretail-angular.scss  # Angular Material global overrides
    ├── _themes.scss        # Client theme variables
    ├── _dark-variables.scss   # Dark mode variables
    ├── _scrollbar.scss     # Scrollbar styles
    └── _normalize.scss     # CSS normalize
```

---

## File Naming Conventions

| File Type | Convention | Example |
|---|---|---|
| Component | `feature-name.component.ts` | `product-list.component.ts` |
| Component template | `feature-name.component.html` | `product-list.component.html` |
| Component styles | `feature-name.component.scss` | `product-list.component.scss` |
| Service | `feature-name.service.ts` | `product.service.ts` |
| Model/Interface | `feature-name.model.ts` | `product.model.ts` |
| Enum | `feature-name.enum.ts` | `product-status.enum.ts` |
| Guard | `feature-name.guard.ts` | `auth.guard.ts` |
| Interceptor | `feature-name.interceptor.ts` | `auth.interceptor.ts` |
| Pipe | `feature-name.pipe.ts` | `currency-format.pipe.ts` |
| Routes | `feature-name.routes.ts` | `product.routes.ts` |
| Directive | `feature-name.directive.ts` | `click-outside.directive.ts` |

---

## Class Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Component | `PascalCase + Component` | `ProductListComponent` |
| Service | `PascalCase + Service` | `ProductService` |
| Interface/Model | `PascalCase` (no prefix) | `Product`, `CreateProductDto` |
| Enum | `PascalCase` | `ProductStatus` |
| Guard | `camelCase + Guard` (function) | `authGuard` |
| Interceptor | `camelCase + Interceptor` (function) | `authInterceptor` |
| Pipe | `PascalCase + Pipe` | `CurrencyFormatPipe` |
| Directive | `PascalCase + Directive` | `ClickOutsideDirective` |

---

## When to Put Code Where

### `core/services/` vs `shared/components/`

- **`core/services/`** → Anything that talks to an API or is a singleton business service
- **`shared/components/`** → Any UI component used in more than one feature page
- **`pages/feature/`** → Components used only inside that specific feature

### Model vs Interface

- Use `interface` for data shapes that come from APIs
- Use `type` for unions, mapped types, utility types
- Suffix DTOs with `Dto`: `CreateProductDto`, `UpdateProductDto`

---

## Barrel Files (index.ts)

Only create `index.ts` barrel files in `shared/components/` subdirectories that are imported frequently.

```typescript
// src/app/shared/components/form-field/index.ts
export * from './form-field.component';
export * from './interfaces/form-field.interface';
```
