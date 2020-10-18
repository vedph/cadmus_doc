# Adding Parts

When adding parts, you can choose to add them to an existing library, or to a new one. In the latter case, first add the UI and PG libraries as specified below. Please notice that the following naming is just a suggested convention.

These new procedures were defined for the Angular-libraries based Cadmus frontend ([Cadmus shell](https://github.com/vedph/cadmus_shell)). The [old procedures](adding-parts-old.md) still include instructions about building demo components, which are less useful at this stage of development.

## Adding UI Library

1. in your web app, add a **new Angular library** for the editor UI elements: `ng generate library @myrmidon/cadmus-PRJNAME-part-LIBNAME-ui`. Please note that here `@myrmidon` is the NPM user name I'm using for publishing my libraries. Replace it with your own, so that no name clashes can occur. This is the "ui" library. In this library you will place the part and fragment editors. You can remove the sample component and service added by Angular.

2. in the "ui" library **module**, add the typical imports, like this:

```ts
import { CommonModule } from "@angular/common";
import { NgModule } from "@angular/core";
import { FormsModule, ReactiveFormsModule } from "@angular/forms";

// general Cadmus modules
import { CadmusMaterialModule } from "@myrmidon/cadmus-material";
import { CadmusUiModule } from "@myrmidon/cadmus-ui";

// project-specific modules
import { CadmusItineraCoreModule } from "@myrmidon/cadmus-itinera-core";
import { CadmusItineraUiModule } from "@myrmidon/cadmus-itinera-ui";

// a part editor component
import { PersonPartComponent } from "./person-part/person-part.component";

@NgModule({
  declarations: [PersonPartComponent],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    // Cadmus
    CadmusMaterialModule,
    CadmusUiModule,
    // Cadmus itinera
    CadmusItineraCoreModule,
    CadmusItineraUiModule,
  ],
  exports: [PersonPartComponent],
})
export class CadmusItineraPartLtUiModule {}
```

3. edit your "ui" library `package.json` to add **peer dependencies**, like in this sample:

```json
{
  "name": "@myrmidon/cadmus-itinera-part-lt-ui",
  "version": "0.0.1",
  "peerDependencies": {
    "@angular/common": "^10.1.4",
    "@angular/core": "^10.1.4",
    "@myrmidon/cadmus-core": "^0.0.1",
    "@myrmidon/cadmus-material": "^0.0.1",
    "@myrmidon/cadmus-itinera-core": "^0.0.1",
    "@myrmidon/cadmus-itinera-ui": "^0.0.1"
  },
  "dependencies": {
    "tslib": "^2.0.0"
  }
}
```

As you can see, every module imported in the "ui" library is listed here among the peer dependencies.

4. edit your "ui" library `ng-package.json` to add an **UMD module IDs** for each imported modules. If you miss any of these, you will get some warnings during the compilation. Example:

```json
{
  "$schema": "../../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../../dist/myrmidon/cadmus-itinera-part-lt-ui",
  "lib": {
    "entryFile": "src/public-api.ts",
    "umdModuleIds": {
      "ngx-monaco-editor": "ngx-monaco-editor",
      "@myrmidon/cadmus-api": "@myrmidon/cadmus-api",
      "@myrmidon/cadmus-material": "@myrmidon/cadmus-material",
      "@myrmidon/cadmus-ui": "@myrmidon/cadmus-ui",
      "@myrmidon/cadmus-itinera-core": "@myrmidon/cadmus-itinera-core",
      "@myrmidon/cadmus-itinera-ui": "@myrmidon/cadmus-itinera-ui"
    }
  }
}
```

## Adding PG Library

1. in your web app, add a **new Angular library** for the editor features ("pages") elements: `ng generate library @myrmidon/cadmus-PRJNAME-part-LIBNAME-pg`. This is the "pg" library. You can remove the sample component and service added by Angular. In this library, every page wraps the dumb UI component into a component which has a corresponding Akita's state, and gets its data pushed via observables. Also, each page has a route to itself. The app module routes will just include a new route entry, representing the base route for all the routes defined for the new library module: customize it as required. For instance, here is the route to the general parts library:

2. add to your web app `app.module` the "root" route to the "pg" library module, like in this sample (where the root route to that module is named `itinera-lt`):

```ts
{
    path: 'items/:iid/itinera-lt',
    loadChildren: () =>
    import('@myrmidon/cadmus-itinera-part-lt-pg').then(
        (module) => module.CadmusItineraPartLtPgModule
    ),
    canActivate: [AuthGuardService],
},
```

Note that you must not explicitly import the target module into your app module, as this is lazily loaded.

3. edit your "ui" library `package.json` to add **peer dependencies**, like in this sample:

```json
{
  "name": "@myrmidon/cadmus-itinera-part-lt-pg",
  "version": "0.0.1",
  "peerDependencies": {
    "@angular/common": "^10.1.4",
    "@angular/core": "^10.1.4",
    "@angular/forms": "^10.1.4",
    "@angular/router": "^10.1.4",
    "@myrmidon/cadmus-material": "^0.0.1",
    "@myrmidon/cadmus-ui-pg": "^0.0.1",
    "@myrmidon/cadmus-itinera-part-lt-ui": "^0.0.1"
  },
  "dependencies": {
    "tslib": "^2.0.0"
  }
}
```

4. edit your "ui" library `ng-package.json` to add an **UMD module IDs** for each imported modules. If you miss any of these, you will get some warnings during the compilation. Example:

```json
{
  "$schema": "../../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../../dist/myrmidon/cadmus-itinera-part-lt-pg",
  "lib": {
    "entryFile": "src/public-api.ts",
    "umdModuleIds": {
      "@datorama/akita": "@datorama/akita",
      "@myrmidon/cadmus-api": "@myrmidon/cadmus-api",
      "@myrmidon/cadmus-core": "@myrmidon/cadmus-core",
      "@myrmidon/cadmus-material": "@myrmidon/cadmus-material",
      "@myrmidon/cadmus-state": "@myrmidon/cadmus-state",
      "@myrmidon/cadmus-ui-pg": "@myrmidon/cadmus-ui-pg",
      "@myrmidon/cadmus-itinera-part-lt-ui": "@myrmidon/cadmus-itinera-part-lt-ui"
    }
  }
}
```

## Adding a Part to the UI Library

1. add the **part model** (derived from `Part`), its type ID constant, and its JSON schema constant to `<part>.ts` (e.g. `note-part.ts`). Remember to add the new file to the exports of the "barrel" `public-api.ts` file in the module. You can use a template like this (replace `__PROJECT__` with the project's name, e.g. `itinera`; and `__NAME__` with your part's name, without the `Part` suffix; e.g. `Note`, adjusting case where required):

```ts
import { Part } from '@myrmidon/cadmus-core';

/**
 * The __NAME__ part model.
 */
export interface __NAME__Part extends Part {
  // TODO: add properties
}

/**
 * The type ID used to identify the __NAME__Part type.
 */
export const __NAME___PART_TYPEID = 'it.vedph.__PROJECT__.__NAME__';

/**
 * JSON schema for the __NAME__ part. This is used in the editor demo.
 * You can use the JSON schema tool at https://jsonschema.net/.
 */
export const __NAME___PART_SCHEMA = {
  $schema: 'http://json-schema.org/draft-07/schema#',
  $id:
    'www.vedph.it/cadmus/parts/__PROJECT__/__LIB__' +
    __NAME___PART_TYPEID +
    '.json',
  type: 'object',
  title: '__NAME__Part',
  required: [
    'id',
    'itemId',
    'typeId',
    'timeCreated',
    'creatorId',
    'timeModified',
    'userId',
    // TODO: add other required properties here...
  ],
  properties: {
    timeCreated: {
      type: 'string',
      pattern: '^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$',
    },
    creatorId: {
      type: 'string',
    },
    timeModified: {
      type: 'string',
      pattern: '^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$',
    },
    userId: {
      type: 'string',
    },
    id: {
      type: 'string',
      pattern: '^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$',
    },
    itemId: {
      type: 'string',
      pattern: '^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$',
    },
    typeId: {
      type: 'string',
      pattern: '^[a-z][-0-9a-z._]*$',
    },
    roleId: {
      type: ['string', 'null'],
      pattern: '^([a-z][-0-9a-z._]*)?$',
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

2. add a **part editor dumb component** named after the part (e.g. `ng g component note-part` for `NotePartComponent` after `NotePart`), and extending `ModelEditorComponentBase<T>` where `T` is the part's type:
   1. in the _constructor_, instantiate its "root" form group (named `form`), filling it with the required controls.
   2. eventually add _thesaurus_ entries properties for binding, populating them by overriding `onThesauriSet` (`protected onThesauriSet() {}`).
   3. implement `OnInit` calling `this.initEditor();` in it.
   4. (from _model to form_): implement `onModelSet` (`protected onModelSet(model: YourModel)`) by calling an `updateForm(model: YourModel)` which either resets the form if the model is falsy, or sets the various form's controls values according to the received model, finally marking the form as pristine.
   5. (from _form to model_): override `getModelFromForm(): YourModel` to get the model from form controls by calling the base class `getModelFromJson`. If this returns null, create a new part object with default values (you just need to set `typeId` for the `Part`'s interface properties); then, fill the part object properties from the form's controls. This merges the inherited properties (from the initial JSON code, if any) with those edited.
   6. build your component's _template_.

Template:

```ts
import { Component, OnInit } from '@angular/core';
import { FormControl, FormBuilder, Validators } from '@angular/forms';

import { ModelEditorComponentBase, DialogService } from '@myrmidon/cadmus-ui';
import { AuthService } from '@myrmidon/cadmus-api';
import { ThesaurusEntry } from '@myrmidon/cadmus-core';

import { __PARTNAME__Part, __PARTNAME___PART_TYPEID } from '../YOURPARTFILE';

/**
 * __PARTNAME__ editor component.
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
  // public tagEntries: ThesaurusEntry[];

  constructor(authService: AuthService, formBuilder: FormBuilder) {
    super(authService);
    // form
    // TODO build controls and set this.form
  }

  public ngOnInit(): void {
    this.initEditor();
  }

  private updateForm(model: __PARTNAME__Part): void {
    if (!model) {
      this.form.reset();
      return;
    }
    // TODO set controls values from model
    this.form.markAsPristine();
  }

  protected onModelSet(model: __PARTNAME__Part): void {
    this.updateForm(model);
  }

  protected onThesauriSet(): void {
    // TODO set entries from this.thesauri, e.g.:
    // const key = "note-tags";
    // if (this.thesauri && this.thesauri[key]) {
    // this.tagEntries = this.thesauri[key].entries;
    // } else {
    //   this.tagEntries = null;
    // }
    // if not using any thesauri, just remove this function
  }

  protected getModelFromForm(): __PARTNAME__Part {
    let part = this.getModelFromJson();
    if (!part) {
      part = {
        itemId: this.itemId,
        id: null,
        typeId: __PARTNAME___PART_TYPEID,
        roleId: this.roleId,
        timeCreated: new Date(),
        creatorId: null,
        timeModified: new Date(),
        userId: null,
        // TODO default values
      };
    }
    // TODO set part.properties from form controls
    return part;
  }
}
```

Sample code:

```ts
import { Component, OnInit } from '@angular/core';
import { FormControl, FormBuilder, Validators } from '@angular/forms';

import { ModelEditorComponentBase, DialogService } from '@myrmidon/cadmus-ui';
import { AuthService } from '@myrmidon/cadmus-api';
import { ThesaurusEntry } from '@myrmidon/cadmus-core';

import { NotePart, NOTE_PART_TYPEID } from '../YOURPARTFILE';

/**
 * Note part editor component.
 * Thesauri: "note-tags" (optional).
 */
@Component({
  selector: "cadmus-note-part",
  templateUrl: "./note-part.component.html",
  styleUrls: ["./note-part.component.css"],
})
export class NotePartComponent
  extends ModelEditorComponentBase<NotePart>
  implements OnInit {
  public tag: FormControl;
  public tags: FormControl;
  public text: FormControl;

  public tagEntries: ThesaurusEntry[];

  public editorOptions = {
    theme: "vs-light",
    language: "markdown",
    wordWrap: "on",
    // https://github.com/atularen/ngx-monaco-editor/issues/19
    automaticLayout: true,
  };

  constructor(authService: AuthService, formBuilder: FormBuilder) {
    super(authService);
    // form
    this.tag = formBuilder.control(null, Validators.maxLength(100));
    this.tags = formBuilder.control([]);
    this.text = formBuilder.control(null, Validators.required);
    this.form = formBuilder.group({
      tag: this.tag,
      tags: this.tags,
      text: this.text,
    });
  }

  public ngOnInit(): void {
    this.initEditor();
  }

  private updateForm(model: NotePart): void {
    if (!model) {
      this.form.reset();
      return;
    }
    this.tag.setValue(model.tag);
    this.tags.setValue(model.tag);
    this.text.setValue(model.text);
    this.form.markAsPristine();
  }

  protected onModelSet(model: NotePart): void {
    this.updateForm(model);
  }

  protected onThesauriSet(): void {
    const key = "note-tags";
    if (this.thesauri && this.thesauri[key]) {
      this.tagEntries = this.thesauri[key].entries;
    } else {
      this.tagEntries = null;
    }
  }

  protected getModelFromForm(): NotePart {
    let part = this.getModelFromJson();
    if (!part) {
      part = {
        itemId: this.itemId,
        id: null,
        typeId: NOTE_PART_TYPEID,
        roleId: this.roleId,
        timeCreated: new Date(),
        creatorId: null,
        timeModified: new Date(),
        userId: null,
        tag: null,
        text: null,
      };
    }
    part.tag = this.tagEntries ? this.tags.value : this.tag.value;
    part.text = this.text.value ? this.text.value.trim() : null;
    return part;
  }
}
```

```html
<form [formGroup]="form" (submit)="save()">
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

3. ensure the component has been added to its module's `declarations` and `exports`, and to the `public-api.ts` barrel file.

## Adding a Part to the PG Library

Each part editor has its component, and its state management artifacts under the same folder (store, query, and service).

1. add a **part editor feature component** named after the part (e.g. `ng g component note-part-feature` for `NotePartFeatureComponent` after `NotePart`). Ensure that this component is both under the module `declarations` and `exports`, and in the `public-api.ts` barrel file.

2. add the corresponding **route** in the "pg" library's module, e.g.:

```ts
{
  path: `${__NAME___PART_TYPEID}/:pid`,
  pathMatch: 'full',
  component: __NAME__PartFeatureComponent,
  canDeactivate: [PendingChangesGuard]
},
```

3. inside this new component's folder, add a new **store** for your model, named `edit-<partname>-part.store.ts`. Template:

```ts
import { Injectable } from '@angular/core';
import { StoreConfig, Store } from '@datorama/akita';

import {
  EditPartState,
  EditPartStoreApi,
  editPartInitialState,
} from '@myrmidon/cadmus-state';

// TODO: add import from your UI library
// import { __NAME___PART_TYPEID } from '@myrmidon/cadmus-itinera-part-lt-ui';

@Injectable({ providedIn: "root" })
@StoreConfig({ name: __NAME___PART_TYPEID })
export class Edit__NAME__PartStore
  extends Store<EditPartState>
  implements EditPartStoreApi {
  constructor() {
    super(editPartInitialState);
  }

  public setDirty(value: boolean): void {
    this.update({ dirty: value });
  }
  public setSaving(value: boolean): void {
    this.update({ saving: value });
  }
}
```

1. in the same folder, add a new **query** for your model, named `edit-<partname>-part.query.ts`. Template:

```ts
import { Injectable } from '@angular/core';

import { UtilService } from '@myrmidon/cadmus-core';
import { EditPartQueryBase } from '@myrmidon/cadmus-state';

import { Edit__NAME__PartStore } from './edit-__NAME__-part.store';

@Injectable({ providedIn: 'root' })
export class Edit__NAME__PartQuery extends EditPartQueryBase {
  constructor(store: Edit__NAME__PartStore, utilService: UtilService) {
    super(store, utilService);
  }
}
```

4. in the same folder, add a new **service** for your model, named `edit-<partname>-part.service.ts`. Template:

```ts
import { Injectable } from '@angular/core';

import { ItemService, ThesaurusService } from '@myrmidon/cadmus-api';
import { EditPartServiceBase } from '@myrmidon/cadmus-state';

import { Edit__NAME__PartStore } from './edit-__NAME__-part.store';

@Injectable({ providedIn: 'root' })
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

5. implement the feature **editor component** by making it extend `EditPartFeatureBase`, like in this code template:

```ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';

import { ThesauriSet } from '@myrmidon/cadmus-core';
import { MatSnackBar } from '@myrmidon/cadmus-material';
import { EditItemQuery, EditItemService, EditPartFeatureBase } from '@myrmidon/cadmus-state';

import { Edit__NAME__PartService } from './edit-__NAME__-part.service';
import { Edit__NAME__PartQuery } from './edit-__NAME__-part.query';

@Component({
  selector: "cadmus-__NAME__-part-feature",
  templateUrl: "./__NAME__-part-feature.component.html",
  styleUrls: ["./__NAME__-part-feature.component.css"],
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

  ngOnInit() {
    // TODO: select your thesauri if required, e.g.:
    // this.initEditor(['note-tags']);
    this.initEditor(['your', 'thesauri', 'ids', 'here']);
  }
}
```

Define the corresponding HTML template like:

```html
<cadmus-current-item-bar></cadmus-current-item-bar>
<cadmus-__NAME__-part
  [itemId]="itemId"
  [roleId]="roleId"
  [json]="json$ | async"
  (jsonChange)="save($event)"
  [thesauri]="thesauri$ | async"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
></cadmus-__NAME__-part>
```

6. in your app's project `part-editor-keys.ts`, add the mapping for the part just created, like e.g.:

```ts
  // itinera parts
  [PERSON_PART_TYPEID]: {
    part: ITINERA_LT
  },
```

Here, the type ID of the part (from its model in the "ui" library) is mapped to the route prefix constant `ITINERA_LT` = `itinera-lt`, which is the root route to the "pg" library module for the app.
