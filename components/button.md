# Button Component

## Custom Button — `app-button`

**Selector:** `app-button`  
**File:** `src/app/shared/components/button/button/button.component.ts`  
**Library used:** Bootstrap Icons + Angular Material Icon + PrimeNG ProgressSpinner

### Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `icon` | `string` | No | Bootstrap icon name (e.g., `'pencil'` → `bi bi-pencil`) |
| `buttonClass` | `string` | No | CSS class applied to the button (`bt-primary`, `bt-danger`, etc.) |
| `loading` | `boolean` | No | Shows spinner when `true`, disables click |
| `buttonType` | `string` | No | HTML `type` attribute (`button`, `submit`, `reset`) |
| `disabled` | `boolean` | No | Disables the button |

### Outputs

| Output | Type | Description |
|---|---|---|
| `clicked` | `MouseEvent` | Emitted on click (only when not loading) |

### Usage

```html
<!-- Primary action -->
<app-button
  buttonClass="bt-primary"
  icon="plus"
  (clicked)="onCreate($event)">
  Novo Item
</app-button>

<!-- Danger / Delete -->
<app-button
  buttonClass="bt-danger"
  icon="trash"
  [loading]="isDeleting()"
  (clicked)="onDelete($event)">
  Excluir
</app-button>

<!-- Default / Cancel -->
<app-button
  buttonClass="bt-default"
  (clicked)="onCancel($event)">
  Cancelar
</app-button>

<!-- Submit with loading state -->
<app-button
  buttonClass="bt-primary"
  buttonType="submit"
  [loading]="form.pending"
  [disabled]="form.invalid">
  Salvar
</app-button>
```

---

## Button CSS Classes

Defined in `src/app/styles/_buttons.scss`. Use these classes on native `<button>` elements or via `app-button`'s `buttonClass` input.

| Class | Background | Text | Usage |
|---|---|---|---|
| `.bt-primary` | `$alt-primary` (#475bd2) | White | Main CTA, confirm actions |
| `.bt-secondary` | `$secondary` | White | Secondary actions |
| `.bt-default` | Transparent | Gray | Cancel, neutral actions |
| `.bt-danger` | `$primary-danger` | White | Delete, destructive actions |
| `.bt-success` | `$primary-success` | White | Approve, confirm success |
| `.bt-black` | `$primary-black` | White | Special dark actions |

### Base Styles (all variants)

```scss
display: flex;
justify-content: center;
align-items: center;
font-size: 14px;
font-weight: bold;
min-height: 40px;
max-height: 40px;
padding: 10px 14px;
border-radius: 8px;
```

### Button Groups

For adjacent buttons that should connect visually:

```html
<button class="bt-primary left-group-item">Left</button>
<button class="bt-default right-group-item">Right</button>
```

| Class | Effect |
|---|---|
| `.left-group-item` | Removes right and bottom-right radius |
| `.right-group-item` | Removes left and top-right radius |

---

## Rules

- ✅ Use `app-button` for all interactive buttons in feature pages
- ✅ Use `bt-primary` for the main action on a page/form
- ✅ Use `bt-default` for cancel/back actions
- ✅ Always pass `[loading]` when the button triggers an async operation
- ❌ Never use raw `<button>` without a `bt-*` class
- ❌ Never use `[ngClass]` — use `[class]` binding or static class
