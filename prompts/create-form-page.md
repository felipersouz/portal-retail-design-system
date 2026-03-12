# Prompt: Create Form Page (Create/Edit)

Use this prompt template in Cursor Chat to generate a complete create/edit form page.

---

## Prompt Template

```
Create a complete Angular 20 create/edit form page for [ENTITY_NAME] following the portal-retail design system.

**Entity:** [ENTITY_NAME]
**Route:** /[route-path]/create and /[route-path]/edit/:id
**File location:** src/app/pages/[feature-folder]/[feature-name]-form/

**Form fields:**
- [field1]: [type: text | number | email | select | date | textarea] | [validators: required | maxLength(N) | email]
- [field2]: [type] | [validators]
- [field3]: [type] — select options: [option1, option2, option3]

**API endpoints:**
- GET /api/[endpoint]/:id  (for loading in edit mode)
- POST /api/[endpoint]     (create)
- PUT /api/[endpoint]/:id  (update)

**After save:** navigate to /[route-path]

**Requirements:**
1. Single component handles both create and edit via route param `id`
2. Use `computed()` for `isEditMode` and `pageTitle`
3. Use `ChangeDetectionStrategy.OnPush`
4. Use Angular Material form fields with `appearance="outline"`
5. Use `app-button` for submit (bt-primary) and cancel (bt-default)
6. Call `form.markAllAsTouched()` before early return on invalid
7. Show inline `mat-error` for each validation
8. Use `.eretail-form-sm` class on the form element
9. Generate TypeScript interfaces for the entity and DTOs

Fetch Angular Material form field docs using Context7 before generating.
```

---

## Example (filled)

```
Create a complete Angular 20 create/edit form page for Product following the portal-retail design system.

Entity: Product
Route: /product/create and /product/edit/:id
File location: src/app/pages/product/product-form/

Form fields:
- name: text | required, maxLength(100)
- sku: text | required
- price: number | required
- category: select | required — options: Electronics, Clothing, Food
- description: textarea
- active: select | required — options: true/false

API endpoints:
- GET /api/products/:id
- POST /api/products
- PUT /api/products/:id

After save: navigate to /product

Requirements: [same as above]
```
