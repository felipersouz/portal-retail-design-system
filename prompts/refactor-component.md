# Prompt: Refactor & Standardize Component

Use this prompt to ask the AI to refactor any existing component or page to comply with
the design system, Angular 20 patterns, and project rules.

---

## How to use

1. Open the file you want to refactor in the editor
2. Copy the prompt below into the Cursor chat (Agent mode)
3. Fill in the `[COMPONENT TYPE]` and `[ROUTE]` placeholders
4. Send — the AI will read the design system and apply all rules automatically

---

## Prompt template

```
Refactor and standardize the file @[filename] to follow the project design system and best practices.

Component type: [listing page | form page | service | shared component]
Route (if applicable): [e.g. /products]

Before making any changes:
1. Read `guidelines/angular-architecture.md` from felipersouz/portal-retail-design-system
2. Read `guidelines/angular-modern-features.md` from felipersouz/portal-retail-design-system
3. Read `guidelines/rxjs.md` from felipersouz/portal-retail-design-system
4. Read `guidelines/code-quality.md` from felipersouz/portal-retail-design-system
5. Read the matching pattern file:
   - For listing pages → `patterns/listing-page.md`
   - For form pages → `patterns/form-page.md`

Then apply every rule from the design system. Make sure to:

ANGULAR MODERNIZATION
- [ ] Replace constructor injection with inject()
- [ ] Replace @Input() / @Output() with input() / output()
- [ ] Replace @ViewChild / @ViewChildren with viewChild() / viewChildren()
- [ ] Remove standalone: true from the decorator (it is the default)
- [ ] Add ChangeDetectionStrategy.OnPush if missing
- [ ] Replace *ngIf / *ngFor / *ngSwitch with @if / @for / @switch
- [ ] Add track expression to every @for
- [ ] Replace [ngClass] / [ngStyle] with [class] / [style]

RXJS
- [ ] Add takeUntilDestroyed(this.destroyRef) to every subscription
- [ ] Replace manual ngOnDestroy + unsubscribe() with takeUntilDestroyed
- [ ] Add catchError + toast.showErrorToast to every HTTP call
- [ ] Add finalize(() => this.isLoading.set(false)) to every loading flow
- [ ] Flatten nested .subscribe() calls with switchMap / mergeMap / concatMap
- [ ] Replace BehaviorSubject for local state with signal()

LOADING & ERROR STATES
- [ ] Replace p-progress-spinner as sole loading state with p-skeleton
- [ ] Skeleton must match the real layout (one skeleton per field/row, matching height)
- [ ] Add three exclusive states: isLoading → hasError → content
- [ ] Add retry button on error state
- [ ] Add toast.showSuccessToast after every successful mutation (save, delete)
- [ ] Add toast.showErrorToast on every error (load, save, delete)

UI COMPONENTS
- [ ] Replace mat-card with p-card
- [ ] Replace mat-form-field / matInput / mat-select with PrimeNG equivalents
- [ ] Replace raw <button> with <app-button buttonClass="bt-*">
- [ ] Replace raw mat-table with <eretail-dynamic-table>

CODE QUALITY
- [ ] Remove all console.log debug lines
- [ ] Replace any types with proper TypeScript types
- [ ] Apply early return (guard clauses) — remove deep nesting
- [ ] Name all magic numbers/strings as constants

After refactoring, provide:
1. A summary of every change made and the reason
2. A list of any patterns you could NOT apply and why (e.g. missing service method)
```

---

## Quick single-file variant

For a fast refactor without specifying type, use:

```
Refactor @[filename] to comply with the portal-retail design system.
Read guidelines/angular-architecture.md, guidelines/rxjs.md and guidelines/code-quality.md
from felipersouz/portal-retail-design-system before starting.
Apply every rule from those files. Show a summary of changes at the end.
```

---

## Tips

- **Agent mode** works best — it can read multiple design system files in sequence before editing
- Always open the file you want refactored so Cursor has context via `@filename`
- For large files, refactor in sections: first modernize Angular patterns, then RxJS, then UI
- If the component uses a service that does not follow conventions, ask to refactor the service too:
  ```
  Also refactor @[service-file] following guidelines/angular-architecture.md
  ```
