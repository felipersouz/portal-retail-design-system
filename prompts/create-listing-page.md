# Prompt: Create Listing Page

Use this prompt template in Cursor Chat to generate a complete listing page.

---

## Prompt Template

```
Create a complete Angular 20 listing page for [ENTITY_NAME] following the portal-retail design system.

**Entity:** [ENTITY_NAME]
**Route:** /[route-path]
**API endpoint:** GET /api/[endpoint]
**File location:** src/app/pages/[feature-folder]/[feature-name]-list/

**Columns to display:**
- [column1]: [type: null | chips | anchor | defaultWithIcon]
- [column2]: [type]
- [column3]: [type]

**Filter fields:**
- [field1]: [text | select | date]
- [field2]: [text | select]

**Action buttons:**
- Edit: navigate to /[route]/edit/:id
- Delete: call service.delete(id)

**Requirements:**
1. Use `eretail-dynamic-table` component
2. Use `signal<T[]>([])` for data
3. Use `ChangeDetectionStrategy.OnPush`
4. Wrap in `content-wrapper`
5. Show `p-progress-spinner` while loading
6. Follow the listing-page pattern from design system
7. Generate the service method `getAll()` if not exists

Fetch the current API of DynamicTableComponent using Context7 before generating.
```

---

## Example (filled)

```
Create a complete Angular 20 listing page for Supplier following the portal-retail design system.

Entity: Supplier
Route: /supplier
API endpoint: GET /api/suppliers
File location: src/app/pages/supplier/supplier-list/

Columns to display:
- name: null
- cnpj: null  
- status: chips
- createdAt: null

Filter fields:
- name: text
- status: select (options: active/inactive)

Action buttons:
- Edit: navigate to /supplier/edit/:id
- Delete: call supplierService.delete(id)

Requirements: [same as above]
```
