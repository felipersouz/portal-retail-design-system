# Portal Retail — Design System

Central source of truth for the `portal-retail-frontend` Angular project.
Defines design tokens, component usage, architecture patterns, and AI prompt templates.

---

## Structure

```
design-system/
├── tokens/
│   ├── colors.md         # Color tokens, SCSS variables, CSS custom properties
│   ├── typography.md     # Font sizes, weights, utility classes
│   └── spacing.md        # Spacing system, layout patterns
│
├── components/
│   ├── button.md         # app-button + bt-* CSS classes
│   ├── dynamic-table.md  # eretail-dynamic-table (main data table)
│   ├── heading.md        # app-heading (page titles)
│   ├── content-wrapper.md # content-wrapper (page layout)
│   └── stat-card.md      # app-stat-card (KPI metrics)
│
├── patterns/
│   ├── listing-page.md   # Full listing page pattern (TypeScript + HTML)
│   └── form-page.md      # Create/Edit form pattern (TypeScript + HTML)
│
├── guidelines/
│   ├── angular-architecture.md  # Angular 20 rules and patterns
│   ├── file-structure.md        # Folder/file naming conventions
│   └── tooling.md               # CLI commands, build, testing
│
├── prompts/
│   ├── create-listing-page.md   # AI prompt to generate listing pages
│   ├── create-form-page.md      # AI prompt to generate form pages
│   └── create-service.md        # AI prompt to generate services
│
└── .cursor/
    └── rules/
        └── design-system.mdc    # Cursor AI rules for this repo
```

---

## Quick Reference

### Tech Stack

| Layer | Technology |
|---|---|
| Framework | Angular 20 (standalone, signals, OnPush) |
| UI (always first) | PrimeNG 20 |
| UI (fallback only) | Angular Material 20 — only when PrimeNG has no equivalent |
| CSS | Bootstrap 5 (utilities + `bi bi-*` icons only) |
| Forms (static) | Angular Reactive Forms |

### Key Rules

1. `ChangeDetectionStrategy.OnPush` on every component
2. `inject()` function — never constructor injection
3. `input()` / `output()` — never `@Input()` / `@Output()`
4. `signal()` / `computed()` — never `BehaviorSubject` for local state
5. `@if` / `@for` — never `*ngIf` / `*ngFor`
6. `[class]` — never `[ngClass]`
7. Always use `<content-wrapper>` as page root
8. Always use `<app-button>` — never raw `<button>` without `bt-*`
9. Always use `<eretail-dynamic-table>` — never raw `mat-table` in pages

---

## How to use with Cursor AI

This repo is connected via the **GitHub MCP** and **Filesystem MCP** in Cursor.

When asking Cursor to generate code, reference the design system:

```
Create a listing page for [entity] following the design system pattern.
Read design-system/patterns/listing-page.md first.
```

Or use the ready-made prompt templates from the `prompts/` folder.

---

## Contributing

When adding or changing project patterns:

1. Update the relevant file in this repo
2. Commit with a clear message: `docs: update button pattern to include loading state`
3. The AI will automatically use the updated rules on next interaction
