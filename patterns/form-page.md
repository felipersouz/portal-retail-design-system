# Pattern: Form Page (Create / Edit)

The standard pattern for create and edit forms. A single `FormComponent` handles both modes using route params.
All form UI uses **PrimeNG** components only.

---

## TypeScript Template

```typescript
import {
  Component, OnInit, inject, signal,
  ChangeDetectionStrategy, computed
} from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { FormBuilder, Validators, ReactiveFormsModule, AbstractControl } from '@angular/forms';
import { CardModule } from 'primeng/card';
import { InputTextModule } from 'primeng/inputtext';
import { SelectModule } from 'primeng/select';
import { TextareaModule } from 'primeng/textarea';
import { IftaLabelModule } from 'primeng/iftalabel';
import { MessageModule } from 'primeng/message';
import { ContentWrapperComponent } from '@shared/components/content-wrapper/content-wrapper.component';
import { HeadingComponent } from '@shared/components/heading/heading.component';
import { CustomButtonComponent } from '@shared/components/button/button/button.component';
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
  ],
  templateUrl: './feature-form.component.html',
})
export class FeatureFormComponent implements OnInit {
  private service = inject(FeatureService);
  private router = inject(Router);
  private route = inject(ActivatedRoute);
  private fb = inject(FormBuilder);

  // Determine create vs edit mode
  private editId = signal<number | null>(null);
  isEditMode = computed(() => this.editId() !== null);
  pageTitle = computed(() => this.isEditMode() ? 'Edit Feature' : 'New Feature');

  isSaving = signal(false);
  isLoading = signal(false);

  // Select options example
  statusOptions = [
    { label: 'Active', value: 'active' },
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

  // Helper: returns true when the field is invalid AND touched (or form submitted)
  isInvalid(control: AbstractControl | null): boolean {
    return !!control && control.invalid && control.touched;
  }

  private loadFeature(id: number): void {
    this.isLoading.set(true);
    this.service.getById(id).subscribe({
      next: (data) => {
        this.form.patchValue(data);
        this.isLoading.set(false);
      },
      error: () => this.isLoading.set(false),
    });
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

    request$.subscribe({
      next: () => {
        this.isSaving.set(false);
        this.router.navigate(['/feature']);
      },
      error: () => this.isSaving.set(false),
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

  @if (isLoading()) {
    <div class="d-flex justify-content-center p-4">
      <p-progress-spinner />
    </div>
  } @else {
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
              class="w-100">
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
            (clicked)="onCancel()">
            Cancel
          </app-button>

          <app-button
            buttonClass="bt-primary"
            buttonType="submit"
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

---

## Rules

- ✅ A single component handles both create and edit via route param `id`
- ✅ Use `computed()` for derived state like `isEditMode` and `pageTitle`
- ✅ Use `isInvalid(control)` helper to keep templates clean
- ✅ Call `form.markAllAsTouched()` before returning on invalid
- ✅ Always set `[invalid]` binding on PrimeNG inputs for visual feedback
- ✅ Wrap every field in `<p-iftalabel>` for consistent label positioning
- ✅ Show `p-message` errors below each field when touched + invalid
- ✅ Show loading state while fetching data in edit mode
- ❌ Never create separate `CreateComponent` and `EditComponent`
- ❌ Never use `mat-form-field`, `matInput`, `mat-select` — use PrimeNG equivalents
- ❌ Never use `[(ngModel)]` — use Reactive Forms
