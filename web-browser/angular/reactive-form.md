### link
- https://www.cnblogs.com/keatkeat/p/18013921

### 基本概念
- **`FormGroup`**：表示一组表单控件的集合。
- **`FormControl`**：表示单个表单控件（如输入框、复选框等）。
- **`FormArray`**：表示一组动态表单控件的集合。

`FormGroup` 的主要功能是：
1. **管理状态**：跟踪表单控件的值和验证状态。
2. **验证**：可以为整个表单组设置验证器。
3. **嵌套结构**：支持嵌套其他 `FormGroup` 或 `FormArray`，以构建复杂的表单。

### 使用方法
1. **创建 `FormGroup`**
   使用 `FormGroup` 和 `FormControl` 创建表单组。

   ```typescript
   import { Component } from '@angular/core';
   import { FormGroup, FormControl, Validators } from '@angular/forms';

   @Component({
     selector: 'app-root',
     template: `
       <form [formGroup]="form" (ngSubmit)="onSubmit()">
         <label>
           Name:
           <input formControlName="name" />
         </label>
         <label>
           Email:
           <input formControlName="email" />
         </label>
         <button type="submit" [disabled]="form.invalid">Submit</button>
       </form>
     `,
   })
   export class AppComponent {
     form = new FormGroup({
       name: new FormControl('', [Validators.required]),
       email: new FormControl('', [Validators.required, Validators.email]),
     });

     onSubmit() {
       console.log(this.form.value); // 获取表单的值
     }
   }
   ```

2. **访问控件状态**
   - `form.value`：获取表单的当前值。
   - `form.valid`：检查表单是否有效。
   - `form.get('controlName')`：访问特定控件。

3. **嵌套 `FormGroup`**
   可以在 `FormGroup` 中嵌套另一个 `FormGroup`。

   ```typescript
   form = new FormGroup({
     user: new FormGroup({
       name: new FormControl('', Validators.required),
       email: new FormControl('', Validators.email),
     }),
   });
   ```

4. **动态表单**
   可以动态添加或删除控件。

   ```typescript
   form.addControl('newControl', new FormControl(''));
   form.removeControl('newControl');
   ```

### 总结
`FormGroup` 是 Angular 响应式表单的核心，用于管理表单控件的状态和验证。通过组合 `FormControl` 和嵌套的 `FormGroup`，可以轻松构建复杂的表单结构。

### 1. **Dynamic FormControl Creation**
This example demonstrates how to dynamically add or remove `FormControl` instances in a `FormGroup`.

```typescript
import { Component } from '@angular/core';
import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-dynamic-form',
  template: `
    <form [formGroup]="form">
      <div *ngFor="let controlName of controlNames; let i = index">
        <label>
          Field {{ i + 1 }}:
          <input [formControlName]="controlName" />
        </label>
        <button type="button" (click)="removeControl(controlName)">Remove</button>
      </div>
      <button type="button" (click)="addControl()">Add Field</button>
    </form>
    <pre>{{ form.value | json }}</pre>
  `,
})
export class DynamicFormComponent {
  form = new FormGroup({});
  controlNames: string[] = [];

  addControl() {
    const controlName = `field${this.controlNames.length + 1}`;
    this.controlNames.push(controlName);
    this.form.addControl(controlName, new FormControl('', Validators.required));
  }

  removeControl(controlName: string) {
    this.controlNames = this.controlNames.filter(name => name !== controlName);
    this.form.removeControl(controlName);
  }
}
```

### 2. **Custom Validator with FormControl**
This example shows how to use a custom validator to validate a `FormControl`.

```typescript
import { Component } from '@angular/core';
import { FormGroup, FormControl, Validators, AbstractControl } from '@angular/forms';

@Component({
  selector: 'app-custom-validator',
  template: `
    <form [formGroup]="form">
      <label>
        Username:
        <input formControlName="username" />
      </label>
      <div *ngIf="form.get('username')?.hasError('invalidUsername')">
        Username must not contain "admin".
      </div>
    </form>
  `,
})
export class CustomValidatorComponent {
  form = new FormGroup({
    username: new FormControl('', [Validators.required, this.forbiddenUsernameValidator]),
  });

  forbiddenUsernameValidator(control: AbstractControl) {
    const forbidden = /admin/.test(control.value);
    return forbidden ? { invalidUsername: true } : null;
  }
}
```

### 3. **Reactive Updates Between FormControls**
This example demonstrates how to make one `FormControl` dynamically update based on the value of another.

```typescript
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormControl } from '@angular/forms';

@Component({
  selector: 'app-reactive-updates',
  template: `
    <form [formGroup]="form">
      <label>
        First Name:
        <input formControlName="firstName" />
      </label>
      <label>
        Full Name:
        <input formControlName="fullName" readonly />
      </label>
    </form>
    <pre>{{ form.value | json }}</pre>
  `,
})
export class ReactiveUpdatesComponent implements OnInit {
  form = new FormGroup({
    firstName: new FormControl(''),
    fullName: new FormControl({ value: '', disabled: true }),
  });

  ngOnInit() {
    this.form.get('firstName')?.valueChanges.subscribe(value => {
      this.form.get('fullName')?.setValue(`${value} Smith`);
    });
  }
}
```

### 4. **Async Validator with FormControl**
This example demonstrates how to use an asynchronous validator to check the availability of a username.

```typescript
import { Component } from '@angular/core';
import { FormGroup, FormControl } from '@angular/forms';
import { of } from 'rxjs';
import { delay, map } from 'rxjs/operators';

@Component({
  selector: 'app-async-validator',
  template: `
    <form [formGroup]="form">
      <label>
        Username:
        <input formControlName="username" />
      </label>
      <div *ngIf="form.get('username')?.pending">Checking availability...</div>
      <div *ngIf="form.get('username')?.hasError('usernameTaken')">
        Username is already taken.
      </div>
    </form>
  `,
})
export class AsyncValidatorComponent {
  form = new FormGroup({
    username: new FormControl('', null, this.usernameAsyncValidator),
  });

  usernameAsyncValidator(control: FormControl) {
    const takenUsernames = ['user1', 'admin', 'test'];
    return of(takenUsernames.includes(control.value))
      .pipe(
        delay(1000), // Simulate server delay
        map(isTaken => (isTaken ? { usernameTaken: true } : null))
      );
  }
}
```

These examples demonstrate how to handle dynamic forms, custom validation, reactive updates, and asynchronous validation with `FormControl`.

### Erklärung:
1. **Erster Parameter**: Der initiale Wert des FormControls (in diesem Fall ein leerer String `''`).
2. **Zweiter Parameter**: Ein oder mehrere **synchronen Validatoren** (z. B. `Validators.required`, `Validators.minLength`, etc.). In diesem Fall ist er `null`, was bedeutet, dass keine synchronen Validatoren verwendet werden.
3. **Dritter Parameter**: Ein oder mehrere **asynchrone Validatoren** (z. B. `this.usernameAsyncValidator`), die typischerweise serverseitige Überprüfungen durchführen.

### Beispiel mit synchronem Validator:
```typescript
new FormControl('', [Validators.required, Validators.minLength(3)], this.usernameAsyncValidator);
```
Hier wird der zweite Parameter als Array von synchronen Validatoren definiert (`Validators.required` und `Validators.minLength(3)`).
