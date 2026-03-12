# Pattern: Form Page (Create / Edit)

The standard pattern for create and edit forms. A single `FormComponent` handles both modes using route params.

---

## TypeScript Template

```typescript
import {
  Component, OnInit, inject, signal,
  ChangeDetectionStrategy, computed
} from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { FormBuilder, Validators, ReactiveFormsModule } from '@angular/forms';
import { MatCardModule } from '@angular/material/card';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
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
    MatCardModule,
    MatFormFieldModule,
    MatInputModule,
    MatSelectModule,
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
    <mat-card>
      <form [formGroup]="form" (ngSubmit)="onSubmit()" class="eretail-form-sm">

        <label class="text-form-section__label">Basic Info</label>

        <mat-form-field appearance="outline" class="w-100">
          <mat-label>Name</mat-label>
          <input matInput formControlName="name" placeholder="Enter name" />
          @if (form.controls.name.hasError('required')) {
            <mat-error>Name is required</mat-error>
          }
        </mat-form-field>

        <mat-form-field appearance="outline" class="w-100">
          <mat-label>Status</mat-label>
          <mat-select formControlName="status">
            <mat-option value="active">Active</mat-option>
            <mat-option value="inactive">Inactive</mat-option>
          </mat-select>
        </mat-form-field>

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
    </mat-card>
  }
</content-wrapper>
```

---

## Rules

- ✅ A single component handles both create and edit via route param `id`
- ✅ Use `computed()` for derived state like `isEditMode` and `pageTitle`
- ✅ Call `form.markAllAsTouched()` before returning on invalid
- ✅ Always disable the submit button when `form.invalid`
- ✅ Show loading state while fetching data in edit mode
- ❌ Never create separate `CreateComponent` and `EditComponent`
- ❌ Never use `[(ngModel)]` — use Reactive Forms
