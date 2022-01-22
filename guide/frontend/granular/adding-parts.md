# Adding Parts (granular)

- [Adding Parts (granular)](#adding-parts-granular)
  - [Creating PG Library](#creating-pg-library)
  - [Creating Part Library](#creating-part-library)
  - [Creating Part](#creating-part)
  - [Adding UI Editor](#adding-ui-editor)
    - [Generic Part Template](#generic-part-template)
    - [List Part Template](#list-part-template)
  - [Adding PG Wrapper](#adding-pg-wrapper)

In the [traditional approach](../adding-parts.md), parts and fragments specific to a project are hosted in two libraries:

- a UI library, including the editors.
- a PG library, including the wrappers for the editors and their routes.

This was the original approach when building the prototype, as it required less infrastructure while focusing on the core development. Now that several Cadmus-based projects are entering the production stage, a more granular approach is preferred, where each part or fragment is hosted in its own library. While requiring a bit more boilerplate code, this has the advantage of freely combining models from different projects, without having e.g. to import all the codicological models when you just need one of them.

This produces a slightly different layout:

- each part/fragment has a UI library, including both its editor and its editor wrapper.
- a single PG library for the whole project imports all the UI libraries and includes their routes.

## Creating PG Library

The project's PG library imports all the part/fragment editors for that project, and includes their routes.

(1) add a new library to the project, named after the pattern `cadmus-part-<PRJNAME>-pg`; for instance, for the `codicology` project the library will be named `cadmus-part-codicology-pg`:

```bash
ng g library @myrmidon/cadmus-part-codicology-pg --prefix cadmus
```

(2) remove the sample service and component files created by Angular in your new library.

(3) add some metadata in its `package.json` file, e.g.:

```json
{
  ...
  "version": "0.0.1",
  "description": "Cadmus codicology - wrappers for part editors.",
  "keywords": [
    "Cadmus"
  ],
  "homepage": "https://github.com/vedph/cadmus-codicology-shell",
  "repository": {
    "type": "git",
    "url": "https://github.com/vedph/cadmus-codicology-shell"
  },
  "author": {
    "name": "Daniele Fusi"
  },
  ...
}
```

(4) implement the module by adding the child routes for each imported part, e.g.:

```ts
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';

// cadmus
import { CadmusCoreModule, PendingChangesGuard } from '@myrmidon/cadmus-core';
import { CadmusStateModule } from '@myrmidon/cadmus-state';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';
import { CadmusUiPgModule } from '@myrmidon/cadmus-ui-pg';
// parts
import {
  CodBindingsPartFeatureComponent,
  COD_BINDINGS_PART_TYPEID,
} from '@myrmidon/cadmus-part-codicology-bindings';

export const RouterModuleForChild = RouterModule.forChild([
  {
    path: `${COD_BINDINGS_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodBindingsPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
]);

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    // Cadmus
    RouterModuleForChild,
    CadmusCoreModule,
    CadmusStateModule,
    CadmusUiModule,
    CadmusUiPgModule,
  ],
  exports: [],
})
export class CadmusPartCodicologyPgModule {}
```

## Creating Part Library

(1) add a new library to the project, named after the pattern `cadmus-part-<PRJNAME>-<PARTNAME>`; for instance, the `bindings` part for the `codicology` project will be named `cadmus-part-codicology-bindings`:

```bash
ng g library @myrmidon/cadmus-part-codicology-bindings --prefix cadmus
```

(2) remove the sample service and component files created by Angular in your new library.

(3) add some metadata in its `package.json` file as above.

(4) in the library `...module.ts` file, add the typical imports and exports (here `__NAME__` is the part's class name, e.g. `CodBindingsPart`, without the `Part` suffix):

```ts
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';

import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatTabsModule } from '@angular/material/tabs';
import { MatSelectModule } from '@angular/material/select';
import { MatTooltipModule } from '@angular/material/tooltip';

// cadmus
import { CadmusStateModule } from '@myrmidon/cadmus-state';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';
import { CadmusUiPgModule } from '@myrmidon/cadmus-ui-pg';
// bricks: TODO: customize
import { CadmusMatPhysicalSizeModule } from '@myrmidon/cadmus-mat-physical-size';
import { CadmusRefsAssertedChronotopeModule } from '@myrmidon/cadmus-refs-asserted-chronotope';
import { CadmusRefsHistoricalDateModule } from '@myrmidon/cadmus-refs-historical-date';
// locals (UI editor and PG wrapper)
import { __NAME__PartComponent } from './__NAME__-part/__NAME__-part.component';
import { __NAME__PartFeatureComponent } from './__NAME__-part-feature/__NAME__-part-feature.component';

@NgModule({
  declarations: [
    __NAME__PartComponent,
    __NAME__PartFeatureComponent
  ],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    // material
    MatButtonModule,
    MatCardModule,
    MatIconModule,
    MatInputModule,
    MatSelectModule,
    MatTabsModule,
    MatTooltipModule,
    // Cadmus
    CadmusStateModule,
    CadmusUiModule,
    CadmusUiPgModule,
    CadmusMatPhysicalSizeModule,
    CadmusRefsAssertedChronotopeModule,
    CadmusRefsHistoricalDateModule,
  ],
  exports: [
    __NAME__PartComponent,
    __NAME__PartFeatureComponent
  ],
})
export class CadmusPart__PRJ____NAME__Module {}
```

## Creating Part

(1) under `src/lib`, add the **part model** file, named `<PARTNAME>.ts` (e.g. `cod-bindings-part.ts`), like this:

```ts
import { Part } from "@myrmidon/cadmus-core";

/**
 * The __NAME__ part model.
 */
export interface __NAME__Part extends Part {
  // TODO: add properties
}

/**
 * The type ID used to identify the __NAME__Part type.
 */
export const __NAME___PART_TYPEID = "it.vedph.__PRJ__.__NAME__";

/**
 * JSON schema for the __NAME__ part.
 * You can use the JSON schema tool at https://jsonschema.net/.
 */
export const __NAME___PART_SCHEMA = {
  $schema: "http://json-schema.org/draft-07/schema#",
  $id:
    "www.vedph.it/cadmus/parts/__PRJ__/__LIB__/" +
    __NAME___PART_TYPEID +
    ".json",
  type: "object",
  title: "__NAME__Part",
  required: [
    "id",
    "itemId",
    "typeId",
    "timeCreated",
    "creatorId",
    "timeModified",
    "userId",
    // TODO: add other required properties here...
  ],
  properties: {
    timeCreated: {
      type: "string",
      pattern: "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$",
    },
    creatorId: {
      type: "string",
    },
    timeModified: {
      type: "string",
      pattern: "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$",
    },
    userId: {
      type: "string",
    },
    id: {
      type: "string",
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
    },
    itemId: {
      type: "string",
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
    },
    typeId: {
      type: "string",
      pattern: "^[a-z][-0-9a-z._]*$",
    },
    roleId: {
      type: ["string", "null"],
      pattern: "^([a-z][-0-9a-z._]*)?$",
    },

    // TODO: add properties and fill the "required" array as needed
  },
};
```

If you want to infer a schema in the [JSON schema tool](https://jsonschema.net/), which is usually the quickest way of writing the schema, you can use this JSON template, adding your model's properties to it:

```json
{
  "id": "009dcbd9-b1f1-4dc2-845d-1d9c88c83269",
  "itemId": "2c2eadb7-1972-4415-9a43-b8036b6fa685",
  "typeId": "it.vedph.thetype",
  "roleId": null,
  "timeCreated": "2019-11-29T16:48:49.694Z",
  "creatorId": "zeus",
  "timeModified": "2019-11-29T16:48:49.694Z",
  "userId": "zeus",
  "TODO": "add properties here"
}
```

(2) add the new file to the exports of the "barrel" `public-api.ts` file in the module, like `export * from './lib/<NAME>-part';`.

## Adding UI Editor

(1) in `src/lib`, add a **part editor dumb component** named after the part (e.g. `ng g component note-part` for `NotePartComponent` after `NotePart`), and extending `ModelEditorComponentBase<T>` where `T` is the part's type:

1. in the _constructor_, instantiate its "root" form group (named `form`), filling it with the required controls.
2. eventually add _thesaurus_ entries properties for binding, populating them by overriding `onThesauriSet` (`protected onThesauriSet() {}`).
3. implement `OnInit` calling `this.initEditor();` in it.
4. (from _model to form_): implement `onModelSet` (`protected onModelSet(model: YourModel)`) by calling an `updateForm(model: YourModel)` which either resets the form if the model is falsy, or sets the various form's controls values according to the received model, finally marking the form as pristine.
5. (from _form to model_): override `getModelFromForm(): YourModel` to get the model from form controls.
6. build your component's _template_.

Here we usually have two cases: a generic part, or a part consisting only of a list of entities. Two different templates are provided here.

### Generic Part Template

(1) write code and HTML template:

```ts
import { Component, OnInit } from "@angular/core";
import { FormControl, FormBuilder, Validators } from "@angular/forms";

import { deepCopy } from '@myrmidon/ng-tools';
import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { ModelEditorComponentBase } from "@myrmidon/cadmus-ui";
import { ThesaurusEntry } from "@myrmidon/cadmus-core";

import { __PARTNAME__Part, __PARTNAME___PART_TYPEID } from "../YOURPARTFILE";

/**
 * __PARTNAME__ part editor component.
 * Thesauri: TODO thesauri names and optionality
 */
@Component({
  selector: "cadmus-__PARTNAME__-part",
  templateUrl: "./__PARTNAME__-part.component.html",
  styleUrls: ["./__PARTNAME__-part.component.css"],
})
export class __PARTNAME__PartComponent
  extends ModelEditorComponentBase<__PARTNAME__Part>
  implements OnInit {
  // TODO form controls (form: FormGroup is inherited)

  // TODO thesauri entries, e.g.:
  // public tagEntries: ThesaurusEntry[] | undefined;

  constructor(authService: AuthJwtService, formBuilder: FormBuilder) {
    super(authService);
    // form
    // TODO build controls and set this.form
  }

  public ngOnInit(): void {
    this.initEditor();
  }

  private updateForm(model: __PARTNAME__Part): void {
    if (!model) {
      this.form!.reset();
      return;
    }
    // TODO set controls values from model
    this.form!.markAsPristine();
  }

  protected onModelSet(model: __PARTNAME__Part): void {
    this.updateForm(deepCopy(model));
  }

  protected override onThesauriSet(): void {
    // TODO set entries from this.thesauri, e.g.:
    // const key = 'note-tags';
    // if (this.thesauri && this.thesauri[key]) {
    // this.tagEntries = this.thesauri[key].entries;
    // } else {
    //   this.tagEntries = undefined;
    // }
    // if not using any thesauri, just remove this function
  }

  protected getModelFromForm(): __PARTNAME__Part {
    let part = this.model;
    if (!part) {
      part = {
        itemId: this.itemId || '',
        id: '',
        typeId: __PARTNAME___PART_TYPEID,
        roleId: this.roleId,
        timeCreated: new Date(),
        creatorId: '',
        timeModified: new Date(),
        userId: '',
        // TODO default values
      };
    }
    // TODO set part.properties from form controls
    return part;
  }
}
```

HTML template:

```html
<form [formGroup]="form!" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>__NAME__ Part</mat-card-title>
    </mat-card-header>
    <mat-card-content> TODO: your template here... </mat-card-content>
    <mat-card-actions>
      <cadmus-close-save-buttons
        [form]="form"
        [noSave]="userLevel < 2"
        (closeRequest)="close()"
      ></cadmus-close-save-buttons>
    </mat-card-actions>
  </mat-card>
</form>
```

(2) ensure the component has been added to its module's `declarations` and `exports`, and to the `public-api.ts` barrel file.

### List Part Template

(1) write code and HTML template:

```ts
import { Component, OnInit } from "@angular/core";
import { FormControl, FormBuilder, Validators } from "@angular/forms";
import { take } from "rxjs/operators";

import { deepCopy, NgToolsValidators } from '@myrmidon/ng-tools';
import { DialogService } from '@myrmidon/ng-mat-tools';
import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { ModelEditorComponentBase } from "@myrmidon/cadmus-ui";
import { ThesaurusEntry } from "@myrmidon/cadmus-core";
import {
  __MODEL__,
  __MODEL__sPart,
  __MODEL__S_PART_TYPEID,
} from "../__MODEL__-part";

/**
 * __MODEL__sPart editor component.
 * Thesauri: TODO.
 */
@Component({
  selector: "tgr-__MODEL__s-part",
  templateUrl: "./__MODEL__s-part.component.html",
  styleUrls: ["./__MODEL__s-part.component.css"],
})
export class __MODEL__sPartComponent
  extends ModelEditorComponentBase<__MODEL__sPart>
  implements OnInit {
  private _editedIndex: number;

  public tabIndex: number;
  public edited__NAME__: __MODEL__ | undefined;

  // TODO: thesaurus entries
  // public tagEntries: ThesaurusEntry[] | undefined;

  public entries: FormControl;

  constructor(
    authService: AuthJwtService,
    formBuilder: FormBuilder,
    private _dialogService: DialogService
  ) {
    super(authService);
    this._editedIndex = -1;
    this.tabIndex = 0;
    // form
    this.entries = formBuilder.control([],
      NgToolsValidators.strictMinLengthValidator(1));
    this.form = formBuilder.group({
      entries: this.entries,
    });
  }

  public ngOnInit(): void {
    this.initEditor();
  }

  private updateForm(model: __MODEL__sPart): void {
    if (!model) {
      this.form!.reset();
      return;
    }
    this.entries.setValue(model.entries || []);
    this.form!.markAsPristine();
  }

  protected onModelSet(model: __MODEL__sPart): void {
    this.updateForm(deepCopy(model));
  }

  protected override onThesauriSet(): void {
    // TODO: thesauri e.g.:
    // let key = 'ms-materials';
    // if (this.thesauri && this.thesauri[key]) {
    //   this.matEntries = this.thesauri[key].entries;
    // } else {
    //   this.matEntries = undefined;
    // }
  }

  protected getModelFromForm(): __MODEL__sPart {
    let part = this.model;
    if (!part) {
      part = {
        itemId: this.itemId || '',
        id: "",
        typeId: __MODEL__S_PART_TYPEID,
        roleId: this.roleId,
        timeCreated: new Date(),
        creatorId: "",
        timeModified: new Date(),
        userId: "",
        entries: [],
      };
    }
    part.entries = this.entries.value || [];
    return part;
  }

  public add__NAME__(): void {
    const entry: __MODEL__ = {
      // TODO set properties
    };
    this.entries.setValue([...this.entries.value, entry]);
    this.edit__NAME__(this.entries.value.length - 1);
  }

  public edit__NAME__(index: number): void {
    if (index < 0) {
      this._editedIndex = -1;
      this.tabIndex = 0;
      this.edited__NAME__ = undefined;
    } else {
      this._editedIndex = index;
      this.edited__NAME__ = this.entries.value[index];
      setTimeout(() => {
        this.tabIndex = 1;
      }, 300);
    }
  }

  public on__NAME__Save(entry: __MODEL__): void {
    this.entries.setValue(
      this.entries.value.map((e: __MODEL__, i: number) =>
        i === this._editedIndex ? entry : e
      )
    );
    this.edit__NAME__(-1);
  }

  public on__NAME__Close(): void {
    this.edit__NAME__(-1);
  }

  public delete__NAME__(index: number): void {
    this._dialogService
      .confirm("Confirmation", "Delete entry?")
      .pipe(take(1))
      .subscribe((yes) => {
        if (yes) {
          const entries = [...this.entries.value];
          entries.splice(index, 1);
          this.entries.setValue(entries);
        }
      });
  }

  public move__NAME__Up(index: number): void {
    if (index < 1) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index - 1, 0, entry);
    this.entries.setValue(entries);
  }

  public move__NAME__Down(index: number): void {
    if (index + 1 >= this.entries.value.length) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index + 1, 0, entry);
    this.entries.setValue(entries);
  }
}
```

HTML:

```html
<form [formGroup]="form!" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>__NAME__s Part</mat-card-title>
    </mat-card-header>
    <mat-card-content>
      <mat-tab-group [(selectedIndex)]="tabIndex">
        <mat-tab label="__NAME__s">
          <div>
            <button
              type="button"
              mat-icon-button
              color="primary"
              (click)="add__NAME__()"
            >
              <mat-icon>add_circle</mat-icon> add __NAME__
            </button>
          </div>
          <table *ngIf="__NAME__s?.value?.length">
            <thead>
              <tr>
                <th></th>
                TODO: add model properties
              </tr>
            </thead>
            <tbody>
              <tr
                *ngFor="
                  let entry of entries?.value;
                  let i = index;
                  let first = first;
                  let last = last
                "
              >
                <td>
                  <button
                    type="button"
                    mat-icon-button
                    color="primary"
                    matTooltip="Edit this __NAME__"
                    (click)="edit__NAME__(i)"
                  >
                    <mat-icon>edit</mat-icon>
                  </button>
                  <button
                    type="button"
                    mat-icon-button
                    matTooltip="Move this __NAME__ up"
                    [disabled]="first"
                    (click)="move__NAME__Up(i)"
                  >
                    <mat-icon>arrow_upward</mat-icon>
                  </button>
                  <button
                    type="button"
                    mat-icon-button
                    matTooltip="Move this __NAME__ down"
                    [disabled]="last"
                    (click)="move__NAME__Down(i)"
                  >
                    <mat-icon>arrow_downward</mat-icon>
                  </button>
                  <button
                    type="button"
                    mat-icon-button
                    color="warn"
                    matTooltip="Delete this __NAME__"
                    (click)="delete__NAME__(i)"
                  >
                    <mat-icon>remove_circle</mat-icon>
                  </button>
                </td>
                TODO: properties
              </tr>
            </tbody>
          </table>
        </mat-tab>

        <mat-tab label="unit" *ngIf="edited__NAME__">
          TODO: editor control with: [model]="edited__NAME__"
          (modelChange)="on__NAME__Save($event)"
          (editorClose)="on__NAME__Close()"
        </mat-tab>
      </mat-tab-group>
    </mat-card-content>
    <mat-card-actions>
      <cadmus-close-save-buttons
        [form]="form"
        [noSave]="userLevel < 2"
        (closeRequest)="close()"
      ></cadmus-close-save-buttons>
    </mat-card-actions>
  </mat-card>
</form>
```

Typically you should edit each single entry in a component (generated with `ng g component <NAME>-editor` where NAME is the model's name, e.g. `cod-binding-editor` for the `cod-bindings-part` component - remember to export it both from the library's module and from its barrel `public-api.ts` file), similar to the following template:

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import {
  FormArray,
  FormBuilder,
  FormControl,
  FormGroup,
  Validators,
} from '@angular/forms';
import { ThesaurusEntry } from '@myrmidon/cadmus-core';

@Component({
  selector: 'tgr-...',
  templateUrl: './...component.html',
  styleUrls: ['./...component.css'],
})
export class __NAME__Component implements OnInit {
  private _model: __TYPE__ | undefined;

  @Input()
  public get model(): __TYPE__ | undefined {
    return this._model;
  }
  public set model(value: __TYPE__ | undefined) {
    this._model = value;
    this.updateForm(value);
  }

  @Output()
  public modelChange: EventEmitter<__TYPE__>;
  @Output()
  public editorClose: EventEmitter<any>;

  public form: FormGroup;
  // TODO: controls

  constructor(formBuilder: FormBuilder) {
    this.modelChange = new EventEmitter<__TYPE__>();
    this.editorClose = new EventEmitter<any>();
    // form
    // TODO: controls
    this.form = formBuilder.group({
      // TODO
    });
  }

  ngOnInit(): void {
    if (this._model) {
      this.updateForm(this._model);
    }
  }

  private updateForm(model: __TYPE__ | undefined): void {
    if (!model) {
      this.form.reset();
      return;
    }

    // TODO set controls values

    this.form.markAsPristine();
  }

  private getModel(): __TYPE__ | null {
    return {
      // TODO get values from controls
    };
  }

  public cancel(): void {
    this.editorClose.emit();
  }

  public save(): void {
    if (this.form.invalid) {
      return;
    }
    const model = this.getModel();
    if (!model) {
      return;
    }
    this.modelChange.emit(model);
  }
}
```

HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  TODO
  <!-- buttons -->
  <div>
    <button
      type="button"
      color="warn"
      mat-icon-button
      matTooltip="Discard changes"
      (click)="cancel()"
    >
      <mat-icon>clear</mat-icon>
    </button>
    <button
      type="submit"
      color="primary"
      mat-icon-button
      matTooltip="Accept changes"
      [disabled]="form.invalid || form.pristine"
    >
      <mat-icon>check_circle</mat-icon>
    </button>
  </div>
</form>
```

(2) ensure the component has been added to its module's `declarations` and `exports`, and to the `public-api.ts` barrel file.

## Adding PG Wrapper

Once you have the part editor, you need its wrapper page, which in turn is linked to a specific route in the context of the project. Such route is defined outside the part's library, in the general PG library for the project.

Each wrapper has its component, and its state management artifacts under the same folder (store, query, and service).

(1) under your library's `src/lib` folder, add a **part editor feature component** named after the part (e.g. `ng g component note-part-feature` for `NotePartFeatureComponent` after `NotePart`).

(2) ensure that this component is both under the module `declarations` and `exports`, and in the `public-api.ts` barrel file.

(3) add the corresponding **route** in the PG library's module of the project's PG library, e.g.:

```ts
{
  path: `${__NAME___PART_TYPEID}/:pid`,
  pathMatch: 'full',
  component: __NAME__PartFeatureComponent,
  canDeactivate: [PendingChangesGuard]
},
```

Note that you should end with this kind of code when using the router module in a library:

```ts
// https://github.com/ng-packagr/ng-packagr/issues/778
export const RouterModuleForChild = RouterModule.forChild([
  {
    path: `${MSUNITS_PART_TYPEID}/:pid`,
    pathMatch: "full",
    component: MsUnitsPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  // ...
]);

@NgModule({
  declarations: [
    // ...
  ],
  imports: [
    // ...
    RouterModuleForChild,
  ],
  exports: [
    // ...
  ],
})
export class CadmusTgrPartMsPgModule {}
```

(4) inside this new component's folder, add a new **store** for your model, named `edit-<partname>-part.store.ts`. Template:

```ts
import { Injectable } from "@angular/core";
import { StoreConfig, Store } from "@datorama/akita";

import {
  EditPartState,
  EditPartStoreApi
} from "@myrmidon/cadmus-state";

// TODO: add import from your UI library
// import { __NAME___PART_TYPEID } from '@myrmidon/cadmus-itinera-part-lt-ui';

@Injectable({ providedIn: "root" })
@StoreConfig({ name: __NAME___PART_TYPEID })
export class Edit__NAME__PartStore
  extends Store<EditPartState>
  implements EditPartStoreApi {
  constructor() {
    super({});
  }

  public setDirty(value: boolean): void {
    this.update({ dirty: value });
  }
  public setSaving(value: boolean): void {
    this.update({ saving: value });
  }
}
```

(5) in the same folder, add a new **query** for your model, named `edit-<partname>-part.query.ts`. Template:

```ts
import { Injectable } from "@angular/core";
import { EditPartQueryBase } from "@myrmidon/cadmus-state";
import { Edit__NAME__PartStore } from "./edit-__NAME__-part.store";

@Injectable({ providedIn: "root" })
export class Edit__NAME__PartQuery extends EditPartQueryBase {
  constructor(store: Edit__NAME__PartStore) {
    super(store);
  }
}
```

(6) in the same folder, add a new **service** for your model, named `edit-<partname>-part.service.ts`. Template:

```ts
import { Injectable } from "@angular/core";
import { ItemService, ThesaurusService } from "@myrmidon/cadmus-api";
import { EditPartServiceBase } from "@myrmidon/cadmus-state";
import { Edit__NAME__PartStore } from "./edit-__NAME__-part.store";

@Injectable({ providedIn: "root" })
export class Edit__NAME__PartService extends EditPartServiceBase {
  constructor(
    editPartStore: Edit__NAME__PartStore,
    itemService: ItemService,
    thesaurusService: ThesaurusService
  ) {
    super(itemService, thesaurusService);
    this.store = editPartStore;
  }
}
```

(7) implement the feature **editor component** by making it extend `EditPartFeatureBase`, like in this code template:

```ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';
import { MatSnackBar } from '@angular/material/snack-bar';

import {
  EditItemQuery,
  EditItemService,
  EditPartFeatureBase,
} from '@myrmidon/cadmus-state';

import { Edit__NAME__PartService } from './edit-__NAME__-part.service';
import { Edit__NAME__PartQuery } from './edit-__NAME__-part.query';

@Component({
  selector: 'cadmus-__NAME__-part-feature',
  templateUrl: './__NAME__-part-feature.component.html',
  styleUrls: ['./__NAME__-part-feature.component.css'],
})
export class __NAME__PartFeatureComponent
  extends EditPartFeatureBase
  implements OnInit {
  constructor(
    router: Router,
    route: ActivatedRoute,
    snackbar: MatSnackBar,
    editPartQuery: Edit__NAME__PartQuery,
    editPartService: Edit__NAME__PartService,
    editItemQuery: EditItemQuery,
    editItemService: EditItemService
  ) {
    super(
      router,
      route,
      snackbar,
      editPartQuery,
      editPartService,
      editItemQuery,
      editItemService
    );
  }

  public ngOnInit(): void {
    // TODO: select your thesauri if required, e.g.:
    // this.initEditor(['note-tags']);
    this.initEditor(['your', 'thesauri', 'ids', 'here']);
  }
}
```

HTML template:

```html
<cadmus-current-item-bar></cadmus-current-item-bar>
<cadmus-__NAME__-part
  [itemId]="itemId"
  [roleId]="roleId"
  [model]="$any(part$ | async)"
  (modelChange)="save($event)"
  [thesauri]="(thesauri$ | async) || undefined"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
></cadmus-__NAME__-part>
```

(8) in your app's project `part-editor-keys.ts`, add the mapping for the part just created, like e.g.:

```ts
  // itinera parts
  [PERSON_PART_TYPEID]: {
    part: ITINERA_LT
  },
```

Here, the type ID of the part (from its model in the "ui" library) is mapped to the route prefix constant `ITINERA_LT` = `itinera-lt`, which is the root route to the "pg" library module for the app.
