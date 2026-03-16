# Pattern: Form Page (Create / Edit)

The standard pattern for create and edit forms. A single `FormComponent` handles both modes using route params.
All form UI uses **PrimeNG** components only.
Uses skeleton loading (edit mode), `ToastService` for feedback, and inline error state.

---

## TypeScript Template

```typescript
import {
  Component, OnInit, inject, signal,
  ChangeDetectionStrategy, computed, DestroyRef
} from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { FormBuilder, Validators, ReactiveFormsModule, AbstractControl } from '@angular/forms';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { catchError, finalize, of } from 'rxjs';
import { CardModule } from 'primeng/card';
import { InputTextModule } from 'primeng/inputtext';
import { SelectModule } from 'primeng/select';
import { TextareaModule } from 'primeng/textarea';
import { IftaLabelModule } from 'primeng/iftalabel';
import { MessageModule } from 'primeng/message';
import { SkeletonModule } from 'primeng/skeleton';
import { ContentWrapperComponent } from '@shared/components/content-wrapper/content-wrapper.component';
import { HeadingComponent } from '@shared/components/heading/heading.component';
import { CustomButtonComponent } from '@shared/components/button/button/button.component';
import { ToastService } from '@core/helpers/toast.service';
import { FeatureService } from '@core/services/feature.service';
import { Feature } from '@core/models/feature.model';

@Component({
  selector: 'app-feature-form',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    ReactiveFormsModule,
    ContentWrapperComponent,
    HeadingComponent,
    CustomButtonComponent,
    CardModule,
    InputTextModule,
    SelectModule,
    TextareaModule,
    IftaLabelModule,
    MessageModule,
    SkeletonModule,
  ],
  templateUrl: './feature-form.component.html',
})
export class FeatureFormComponent implements OnInit {
  private service    = inject(FeatureService);
  private router     = inject(Router);
  private route      = inject(ActivatedRoute);
  private fb         = inject(FormBuilder);
  private toast      = inject(ToastService);
  private destroyRef = inject(DestroyRef);

  // Create vs Edit mode
  private editId  = signal<number | null>(null);
  isEditMode      = computed(() => this.editId() !== null);
  pageTitle       = computed(() => this.isEditMode() ? 'Edit Feature' : 'New Feature');

  isSaving   = signal(false);
  isLoading  = signal(false);
  hasError   = signal(false);

  statusOptions = [
    { label: 'Active',   value: 'active' },
    { label: 'Inactive', value: 'inactive' },
  ];

  form = this.fb.group({
    name:        ['', [Validators.required, Validators.maxLength(100)]],
    description: [''],
    status:      [null as string | null, Validators.required],
  });

  ngOnInit(): void {
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.editId.set(Number(id));
      this.loadFeature(Number(id));
    }
  }

  // Returns true when the field is invalid AND touched
  isInvalid(control: AbstractControl | null): boolean {
    return !!control && control.invalid && control.touched;
  }

  private loadFeature(id: number): void {
    this.isLoading.set(true);
    this.hasError.set(false);
    this.form.disable();

    this.service.getById(id).pipe(
      catchError(() => {
        this.hasError.set(true);
        this.toast.showErrorToast('Error', 'Failed to load the record. Please try again.');
        return of(null);
      }),
      finalize(() => {
        this.isLoading.set(false);
        this.form.enable();
      }),
      takeUntilDestroyed(this.destroyRef),
    ).subscribe(data => {
      if (data) this.form.patchValue(data);
    });
  }

  onRetry(): void {
    if (this.editId()) this.loadFeature(this.editId()!);
  }

  onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.isSaving.set(true);
    const payload = this.form.getRawValue();

    const request$ = this.isEditMode()
      ? this.service.update(this.editId()!, payload)
      : this.service.create(payload);

    request$.pipe(
      finalize(() => this.isSaving.set(false)),
      takeUntilDestroyed(this.destroyRef),
    ).subscribe({
      next: () => {
        this.toast.showSuccessToast(
          'Success',
          this.isEditMode() ? 'Feature updated successfully.' : 'Feature created successfully.'
        );
        this.router.navigate(['/feature']);
      },
      error: () => {
        this.toast.showErrorToast('Error', 'Failed to save. Please try again.');
      },
    });
  }

  onCancel(): void {
    this.router.navigate(['/feature']);
  }
}
```

---

## HTML Template

```html
<content-wrapper>
  <app-heading [title]="pageTitle()" />

  <!-- ── Loading state: skeleton (edit mode only) ── -->
  @if (isLoading()) {
    <p-card>
      <p-skeleton width="30%" height="1rem" styleClass="mb-4" />

      <p-skeleton width="100%" height="2.75rem" styleClass="mb-3" />
      <p-skeleton width="100%" height="5rem"    styleClass="mb-3" />
      <p-skeleton width="100%" height="2.75rem" styleClass="mb-3" />

      <div class="d-flex justify-content-end gap-2 mt-3">
        <p-skeleton width="6rem" height="2.25rem" />
        <p-skeleton width="8rem" height="2.25rem" />
      </div>
    </p-card>
  }

  <!-- ── Error state: inline message with retry ── -->
  @else if (hasError()) {
    <p-card>
      <div class="d-flex flex-column align-items-center gap-3 py-4">
        <p-message
          severity="error"
          text="Failed to load the record. Check your connection and try again." />
        <div class="d-flex gap-2">
          <app-button buttonClass="bt-default" data-cy="feature-back-btn" (clicked)="onCancel()">
            Back
          </app-button>
          <app-button buttonClass="bt-primary" icon="arrow-clockwise" data-cy="feature-retry-btn" (clicked)="onRetry()">
            Try again
          </app-button>
        </div>
      </div>
    </p-card>
  }

  <!-- ── Success state: form ── -->
  @else {
    <p-card>
      <form [formGroup]="form" (ngSubmit)="onSubmit()" class="eretail-form-sm">

        <label class="text-form-section__label">Basic Info</label>

        <!-- Text input -->
        <div class="flex flex-col gap-1 mb-3">
          <p-iftalabel>
            <input
              pInputText
              id="name"
              formControlName="name"
              class="w-100"
              data-cy="feature-name-input"
              [invalid]="isInvalid(form.controls.name)" />
            <label for="name">Name</label>
          </p-iftalabel>
          @if (form.controls.name.touched && form.controls.name.hasError('required')) {
            <p-message severity="error" size="small" variant="simple">Name is required</p-message>
          }
          @if (form.controls.name.touched && form.controls.name.hasError('maxlength')) {
            <p-message severity="error" size="small" variant="simple">Name must be at most 100 characters</p-message>
          }
        </div>

        <!-- Textarea -->
        <div class="flex flex-col gap-1 mb-3">
          <p-iftalabel>
            <textarea
              pTextarea
              id="description"
              formControlName="description"
              rows="3"
              class="w-100"
              data-cy="feature-description-textarea">
            </textarea>
            <label for="description">Description</label>
          </p-iftalabel>
        </div>

        <!-- Select / Dropdown -->
        <div class="flex flex-col gap-1 mb-3">
          <p-iftalabel>
            <p-select
              inputId="status"
              formControlName="status"
              [options]="statusOptions"
              optionLabel="label"
              optionValue="value"
              placeholder="Select status"
              class="w-100"
              [inputAttrs]="{ 'data-cy': 'feature-status-select' }"
              [invalid]="isInvalid(form.controls.status)" />
            <label for="status">Status</label>
          </p-iftalabel>
          @if (form.controls.status.touched && form.controls.status.hasError('required')) {
            <p-message severity="error" size="small" variant="simple">Status is required</p-message>
          }
        </div>

        <div class="d-flex gap-2 justify-content-end mt-3">
          <app-button
            buttonClass="bt-default"
            data-cy="feature-cancel-btn"
            (clicked)="onCancel()">
            Cancel
          </app-button>

          <app-button
            buttonClass="bt-primary"
            buttonType="submit"
            data-cy="feature-submit-btn"
            [loading]="isSaving()"
            [disabled]="form.invalid">
            {{ isEditMode() ? 'Save Changes' : 'Create' }}
          </app-button>
        </div>

      </form>
    </p-card>
  }
</content-wrapper>
```

---

## Skeleton for Form Fields

Match the number and size of skeletons to the real fields so the layout doesn't shift:

```html
<!-- One skeleton per field, height matches the real input -->
<p-skeleton width="100%" height="2.75rem" styleClass="mb-3" />  <!-- text input -->
<p-skeleton width="100%" height="5rem"    styleClass="mb-3" />  <!-- textarea -->
<p-skeleton width="100%" height="2.75rem" styleClass="mb-3" />  <!-- select -->

<!-- Action buttons (always in a row at the end) -->
<div class="d-flex justify-content-end gap-2 mt-3">
  <p-skeleton width="6rem" height="2.25rem" />   <!-- Cancel -->
  <p-skeleton width="8rem" height="2.25rem" />   <!-- Save -->
</div>
```

---

## PrimeNG Form Components Reference

| Need | Component | Import |
|---|---|---|
| Text input | `pInputText` directive on `<input>` | `InputTextModule` from `primeng/inputtext` |
| Textarea | `pTextarea` directive on `<textarea>` | `TextareaModule` from `primeng/textarea` |
| Dropdown / Select | `<p-select>` | `SelectModule` from `primeng/select` |
| Number input | `pInputText` with `type="number"` | `InputTextModule` |
| Password | `<p-password>` | `PasswordModule` from `primeng/password` |
| Date picker | `<p-datepicker>` | `DatePickerModule` from `primeng/datepicker` |
| Checkbox | `<p-checkbox>` | `CheckboxModule` from `primeng/checkbox` |
| Radio | `<p-radiobutton>` | `RadioButtonModule` from `primeng/radiobutton` |
| Infield label | `<p-iftalabel>` wrapping input | `IftaLabelModule` from `primeng/iftalabel` |
| Error message | `<p-message severity="error" size="small" variant="simple">` | `MessageModule` from `primeng/message` |
| Card container | `<p-card>` | `CardModule` from `primeng/card` |
| Skeleton | `<p-skeleton>` | `SkeletonModule` from `primeng/skeleton` |

---

## `data-cy` Attribute Convention (Cypress)

Every interactive element must have a `data-cy` attribute for E2E tests.
The format is: **`[feature]-[field]-[element-type]`**

| Element type | Suffix | Example |
|---|---|---|
| `<input pInputText>` | `-input` | `data-cy="feature-name-input"` |
| `<textarea pTextarea>` | `-textarea` | `data-cy="feature-description-textarea"` |
| `<p-select>` | `-select` (via `[inputAttrs]`) | `[inputAttrs]="{ 'data-cy': 'feature-status-select' }"` |
| `<p-datepicker>` | `-datepicker` (via `[inputAttrs]`) | `[inputAttrs]="{ 'data-cy': 'feature-date-datepicker' }"` |
| `<p-checkbox>` | `-checkbox` | `data-cy="feature-active-checkbox"` |
| `<p-password>` | `-password-input` (via `[inputAttrs]`) | `[inputAttrs]="{ 'data-cy': 'feature-password-input' }"` |
| Submit button | `-submit-btn` | `data-cy="feature-submit-btn"` |
| Cancel button | `-cancel-btn` | `data-cy="feature-cancel-btn"` |
| Back button (error state) | `-back-btn` | `data-cy="feature-back-btn"` |
| Retry button (error state) | `-retry-btn` | `data-cy="feature-retry-btn"` |

> **Note:** PrimeNG components that render their own `<input>` internally (e.g. `p-select`,
> `p-datepicker`, `p-password`) do **not** forward `data-cy` as a plain attribute.
> Use `[inputAttrs]="{ 'data-cy': '...' }"` to inject the attribute into the native input element.

---

## Rules

- ✅ Three exclusive states: `isLoading` (skeleton) → `hasError` (inline error) → form
- ✅ Skeleton must mirror the actual form layout (one skeleton per field, matching height)
- ✅ Disable the form with `form.disable()` while loading, re-enable in `finalize()`
- ✅ Call `toast.showSuccessToast` after successful save
- ✅ Call `toast.showErrorToast` on save error AND on load error
- ✅ Show inline `p-message` per field for validation errors (touched + hasError)
- ✅ Show `[invalid]` binding on PrimeNG inputs for visual feedback
- ✅ Use `form.markAllAsTouched()` before returning on invalid submit
- ✅ Use `catchError` + `finalize` in the pipe — never inside subscribe callbacks
- ✅ Use `takeUntilDestroyed(this.destroyRef)` on every subscription
- ✅ A single component handles both create and edit via route param `id`
- ✅ Use `computed()` for `isEditMode` and `pageTitle`
- ✅ Show retry button with back option on page load error
- ❌ Never create separate `CreateComponent` and `EditComponent`
- ❌ Never use `mat-form-field`, `matInput`, `mat-select` — use PrimeNG equivalents
- ❌ Never use `[(ngModel)]` — use Reactive Forms
- ❌ Never show `p-progress-spinner` alone as loading state in forms — use skeleton
- ✅ Every interactive element must have a `data-cy` attribute — format: `[feature]-[field]-[element-type]`
- ✅ PrimeNG components with internal `<input>` (p-select, p-datepicker, p-password) use `[inputAttrs]="{ 'data-cy': '...' }"` — never plain `data-cy` on the component tag