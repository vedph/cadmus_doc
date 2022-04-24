# Adding Fragments

- [Adding Fragments](#adding-fragments)
  - [Adding a Parts/Fragments Libraries](#adding-a-partsfragments-libraries)
  - [Adding Fragment to the UI Library](#adding-fragment-to-the-ui-library)
    - [Fragment with List](#fragment-with-list)
  - [Adding Fragment to the PG Library](#adding-fragment-to-the-pg-library)

## Adding a Parts/Fragments Libraries

Same as [parts](adding-parts.md).

## Adding Fragment to the UI Library

In the `<partgroup>-ui` module:

1. add the fragment _model_ (derived from `Fragment`), its type ID constant, and its JSON schema constant to `<fragment>.ts` (e.g. `comment-fragment.ts`). You can use a template like this (replace `__NAME__` with your part's name, e.g. `Comment`, adjusting case where required):

```ts
import { Fragment } from "@myrmidon/cadmus-core";

/**
 * The __NAME__ layer fragment server model.
 */
export interface __NAME__Fragment extends Fragment {
  // TODO: add properties
}

export const __NAME___FRAGMENT_TYPEID = "fr.it.vedph.__PRJ__.__NAME__";

export const __NAME___FRAGMENT_SCHEMA = {
  definitions: {},
  $schema: "http://json-schema.org/draft-07/schema#",
  $id:
    "www.vedph.it/cadmus/fragments/<PRJ>/" +
    __NAME___FRAGMENT_TYPEID +
    ".json",
  type: "object",
  title: "__NAME__Fragment",
  // TODO: add which properties are required
  required: ["location"],
  properties: {
    location: {
      $id: "#/properties/location",
      type: "string",
    },
    baseText: {
      $id: "#/properties/baseText",
      type: "string",
    },
    // TODO: add properties
  },
};
```

If you want to infer a schema in the [JSON schema tool](https://jsonschema.net/), which is usually the quickest way of writing the schema, you can use this JSON template adding your model's properties to it:

```json
{
  "location": "1.2",
  "baseText": "abc",
  "TODO": "add properties here"
}
```

2. add the export for the new file to the library's "barrel" file `public-api.ts`, e.g. `export * from './lib/<NAME>';`.

3. add a _fragment editor dumb component_ named after the fragment (e.g. `ng g component comment-fragment` for `CommentFragmentComponent` after `CommentFragment`), and extending `ModelEditorComponentBase<T>` where `T` is the fragment's type:
   1. in the _constructor_, instantiate its "root" form group (named `form`), filling it with the required controls.
   2. eventually add _thesaurus_ entries properties for binding, populating them by overriding `onThesauriSet` (`protected onThesauriSet() {}`).
   3. implement `OnInit` calling `this.initEditor();` in it.
   4. (from _model to form_): implement `onModelSet` (`protected onModelSet(model: YourModel)`) by calling an `updateForm(model: YourModel)` which either resets the form if the model is falsy, or sets the various form's controls values according to the received model, finally marking the form as pristine.
   5. (from _form to model_): override `getModelFromForm(): YourModel` to get the model from form controls.
   6. build your component's _template_.

Code template:

```ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormControl, Validators } from '@angular/forms';
import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { Fragment, ThesaurusEntry } from '@myrmidon/cadmus-core';
import { ModelEditorComponentBase } from '@myrmidon/cadmus-ui';
import { deepCopy } from '@myrmidon/ng-tools';
import { __NAME__Fragment } from "../__NAME__-fragment";

/**
 * __NAME__ fragment editor component.
 * Thesauri: TODO...
 */
@Component({
  selector: "cadmus-__NAME__-fragment",
  templateUrl: "./__NAME__-fragment.component.html",
  styleUrls: ["./__NAME__-fragment.component.css"],
})
export class __NAME__FragmentComponent
  extends ModelEditorComponentBase<__NAME__Fragment>
  implements OnInit {
  // TODO: add form controls

  // TODO: add tag entries if required, e.g.:
  // public tagEntries: ThesaurusEntry[] | undefined;

  constructor(authService: AuthJwtService, formBuilder: FormBuilder) {
    super(authService);
    // form
    // TODO: add your controls, e.g.:
    // this.tag = formBuilder.control(null, Validators.maxLength(100));
    this.form = formBuilder.group({
      // TODO: add controls to group, e.g.:
      // tag: this.tag,
    });
  }

  public ngOnInit(): void {
    this.initEditor();
  }

  private updateForm(model: __NAME__Fragment): void {
    if (!model) {
      this.form!.reset();
      return;
    }
    // TODO: set form controls from model, e.g.:
    // this.tag.setValue(model.tag);
    this.form.markAsPristine();
  }

  protected onModelSet(model: __NAME__Fragment): void {
    this.updateForm(deepCopy(model));
  }

  protected onThesauriSet(): void {
    // TODO: set thesaurus entries
    // const key = 'comment-tags';
    // if (this.thesauri && this.thesauri[key]) {
    //   this.tagEntries = this.thesauri[key].entries;
    // } else {
    //   this.tagEntries = undefined;
    // }
  }

  protected getModelFromForm(): __NAME__Fragment {
    return {
      location: this.model?.location ?? "",
      // TODO: set fr properties from form's controls
    };
  }
}
```

Sample HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>textsms</mat-icon>
      </div>
      <mat-card-title>LingTags Fragment {{ model?.location }}</mat-card-title>
      <mat-card-subtitle> {{ model?.baseText }} </mat-card-subtitle>
    </mat-card-header>

    <mat-card-content> TODO: add controls </mat-card-content>

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

4. remember to add the component to the barrel `public-api.ts`, and to its module's `declarations` and `exports`.

### Fragment with List

```ts
import { Component, OnInit } from "@angular/core";
import { FormBuilder, FormControl, Validators } from "@angular/forms";
import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { deepCopy } from '@myrmidon/ng-tools';
import { DialogService, ModelEditorComponentBase } from "@myrmidon/cadmus-ui";
import { take } from "rxjs/operators";

@Component({
  selector: "__PRJ__-__NAME__-fragment",
  templateUrl: "./__NAME__-fragment.component.html",
  styleUrls: ["./__NAME__-fragment.component.css"],
})
export class __NAME__FragmentComponent
  extends ModelEditorComponentBase<__NAME__Fragment>
  implements OnInit {
  private _editedIndex: number;

  public tabIndex: number;
  public editedEntry: __MODEL__ | undefined;

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
    this.entries = formBuilder.control([], Validators.required);
    this.form = formBuilder.group({
      entries: this.entries,
    });
  }

  public ngOnInit(): void {
    this.initEditor();
  }

  private updateForm(model: __NAME__Fragment): void {
    if (!model) {
      this.form.reset();
      return;
    }
    this.entries.setValue(model.entries || []);
    this.form.markAsPristine();
  }

  protected onModelSet(model: __NAME__Fragment): void {
    this.updateForm(deepCopy(model));
  }

  protected onThesauriSet(): void {
    // TODO: thesauri e.g.:
    // let key = 'ms-materials';
    // if (this.thesauri && this.thesauri[key]) {
    //   this.matEntries = this.thesauri[key].entries;
    // } else {
    //   this.matEntries = undefined;
    // }
  }

  protected getModelFromForm(): __NAME__Fragment {
    return {
      location: this.model?.location ?? "",
      entries: this.entries.value?.length ? this.entries.value : undefined,
    };
  }

  public addEntry(): void {
    const entry: __MODEL__ = {
      // TODO set parts
    };
    this.entries.setValue([...this.entries.value, entry]);
    this.editEntry(this.entries.value.length - 1);
  }

  public editEntry(index: number): void {
    if (index < 0) {
      this._editedIndex = -1;
      this.tabIndex = 0;
      this.editedEntry = undefined;
    } else {
      this._editedIndex = index;
      this.editedEntry = this.entries.value[index];
      setTimeout(() => {
        this.tabIndex = 1;
      }, 300);
    }
  }

  public onEntrySave(entry: __MODEL__): void {
    this.entries.setValue(
      this.entries.value.map((e: __MODEL__, i: number) =>
        i === this._editedIndex ? entry : e
      )
    );
    this.editEntry(-1);
  }

  public onEntryClose(): void {
    this.editEntry(-1);
  }

  public deleteEntry(index: number): void {
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

  public moveEntryUp(index: number): void {
    if (index < 1) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index - 1, 0, entry);
    this.entries.setValue(entries);
  }

  public moveEntryDown(index: number): void {
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

HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>textsms</mat-icon>
      </div>
      <mat-card-title>__NAME__ Fragment {{ model?.location }}</mat-card-title>
      <mat-card-subtitle> {{ model?.baseText }} </mat-card-subtitle>
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

## Adding Fragment to the PG Library

In a `<partgroup>-pg` module:

1. add a _fragment editor feature component_ named after the part (e.g. `ng g component comment-fragment-feature` for `CommentFragmentFeatureComponent` after `CommentFragment`), with routing. Each editor has its component, and its state management artifacts under the same folder (store, query, and service). Add the corresponding route in the module file, under the `RouterModuleForChild` constant (which is among the `imports` of the module) e.g.:

```ts
export const RouterModuleForChild = RouterModule.forChild([
  // other routes...
  {
    path: `fragment/:pid/${__NAME___FRAGMENT_TYPEID}/:loc`,
    pathMatch: "full",
    component: __NAME__FragmentFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
]);
```

2. ensure that the component added is both under the module `declarations` (imports and exports) and in `public-api.ts`.

3. inside this new component's folder, add a new **store** for your model, named `edit-<fragmentname>-fragment.store.ts`. Template:

```ts
import { Injectable } from "@angular/core";
import { StoreConfig, Store } from "@datorama/akita";
import {
  EditFragmentState,
  EditFragmentStoreApi,
  editFragmentInitialState,
} from "@myrmidon/cadmus-state";
// change this import as required
import { __NAME___FRAGMENT_TYPEID } from "@myrmidon/cadmus-tgr-part-gr-ui";

@Injectable({ providedIn: "root" })
@StoreConfig({ name: __NAME___FRAGMENT_TYPEID })
export class Edit__NAME__FragmentStore
  extends Store<EditFragmentState>
  implements EditFragmentStoreApi {
  constructor() {
    super(editFragmentInitialState);
  }

  public setDirty(value: boolean): void {
    this.update({ dirty: value });
  }
  public setSaving(value: boolean): void {
    this.update({ saving: value });
  }
}
```

4. in the same folder, add a new **query** for your model, named `edit-<fragmentname>-fragment.query.ts`. Template:

```ts
import { Injectable } from "@angular/core";
import { EditFragmentQueryBase } from "@myrmidon/cadmus-state";
import { Edit__NAME__FragmentStore } from "./edit-__NAME__-fragment.store";

@Injectable({ providedIn: "root" })
export class Edit__NAME__FragmentQuery extends EditFragmentQueryBase {
  constructor(protected store: Edit__NAME__FragmentStore) {
    super(store);
  }
}
```

5. in the same folder, add a new **service** for your model, named `edit-<fragmentname>-fragment.service.ts`. Template:

```ts
import { Injectable } from "@angular/core";
import { ItemService, ThesaurusService } from "@myrmidon/cadmus-api";
import { EditFragmentServiceBase } from "@myrmidon/cadmus-state";
import { Edit__NAME__FragmentStore } from "./edit-__NAME__-fragment.store";

@Injectable({ providedIn: "root" })
export class Edit__NAME__FragmentService extends EditFragmentServiceBase {
  constructor(
    editPartStore: Edit__NAME__FragmentStore,
    itemService: ItemService,
    thesaurusService: ThesaurusService
  ) {
    super(itemService, thesaurusService);
    this.store = editPartStore;
  }
}
```

6. implement the feature editor component by making it extend `EditFragmentFeatureBase`, like in this code template:

```ts
import { Component, OnInit } from "@angular/core";
import { Router, ActivatedRoute } from "@angular/router";
import { Edit__NAME__FragmentQuery } from "./edit-__NAME__-fragment.query";
import { Edit__NAME__FragmentService } from "./edit-__NAME__-fragment.service";
import {
  EditItemQuery,
  EditItemService,
  EditLayerPartQuery,
  EditLayerPartService,
  EditFragmentFeatureBase,
} from "@myrmidon/cadmus-state";
import { LibraryRouteService } from "@myrmidon/cadmus-core";

@Component({
  selector: "cadmus-__NAME__-fragment-feature",
  templateUrl: "./__NAME__-fragment-feature.component.html",
  styleUrls: ["./__NAME__-fragment-feature.component.css"],
})
export class __NAME__FragmentFeatureComponent
  extends EditFragmentFeatureBase
  implements OnInit {
  constructor(
    router: Router,
    route: ActivatedRoute,
    snackbar: MatSnackBar,
    editFrQuery: Edit__NAME__FragmentQuery,
    editFrService: Edit__NAME__FragmentService,
    editItemQuery: EditItemQuery,
    editItemService: EditItemService,
    editLayersQuery: EditLayerPartQuery,
    editLayersService: EditLayerPartService,
    libraryRouteService: LibraryRouteService
  ) {
    super(
      router,
      route,
      snackbar,
      editFrQuery,
      editFrService,
      editItemQuery,
      editItemService,
      editLayersQuery,
      editLayersService,
      libraryRouteService
    );
  }

  ngOnInit() {
    // TODO: add the required thesauri IDs in the initEditor call,
    // like 'comment-tags' in this sample
    this.initEditor(["comment-tags"]);
  }
}
```

Define the corresponding HTML template like:

```html
<cadmus-current-item-bar></cadmus-current-item-bar>
<div class="base-text">
  <cadmus-decorated-token-text
    [baseText]="$any(baseText$ | async)"
    [locations]="[$any(frLoc)]"
  ></cadmus-decorated-token-text>
</div>
<cadmus-__NAME__-fragment
  [model]="$any(fragment$ | async)"
  (modelChange)="save($event)"
  [thesauri]="$any(thesauri$ | async)"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
></cadmus-__NAME__-fragment>
```

7. in your app's project `part-editor-keys.ts`, add the mapping for the fragment just created under the layer part key, like e.g.:

```ts
// layer parts
[TOKEN_TEXT_LAYER_PART_TYPEID]: {
  part: GENERAL,
  fragments: {
    [COMMENT_FRAGMENT_TYPEID]: GENERAL,
    [APPARATUS_FRAGMENT_TYPEID]: PHILOLOGY,
    [LING_TAGS_FRAGMENT_TYPEID]: TGR_GR,
  },
},
```

Here, the type ID of the fragment (from its model in the "ui" library) is mapped to the route prefix constant TGR_GR = `tgr-gr`, which is the root route to the "pg" library module for the app.
