# Adding Fragments

## Adding a Parts/Fragments Libraries

Same as [parts](adding-parts.md).

## Adding Fragment to the UI Library

In the `<partgroup>-ui` module:

1. add the fragment _model_ (derived from `Fragment`), its type ID constant, and its JSON schema constant to `<fragment>.ts` (e.g. `comment-fragment.ts`). Remember to add the new file to the "barrel" `index.ts` in the module. You can use a template like this (replace `__NAME__` with your part's name, e.g. `Comment`, adjusting case where required):

```ts
import { Fragment } from '@myrmidon/cadmus-core';

/**
 * The __NAME__ layer fragment server model.
 */
export interface __NAME__Fragment extends Fragment {
  // TODO: add properties
}

export const __NAME___FRAGMENT_TYPEID = 'fr.it.vedph.__NAME__';

export const __NAME___FRAGMENT_SCHEMA = {
  definitions: {},
  $schema: 'http://json-schema.org/draft-07/schema#',
  $id: 'www.vedph.it/cadmus/fragments/general/' + __NAME___FRAGMENT_TYPEID + '.json',
  type: 'object',
  title: '__NAME__Fragment',
  // TODO: add which properties are required
  required: ['location'],
  properties: {
    location: {
      $id: '#/properties/location',
      type: 'string'
    },
    baseText: {
      $id: '#/properties/baseText',
      type: 'string'
    },
    // TODO: add properties
  }
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

2. add a _fragment editor dumb component_ named after the fragment (e.g. `ng g component comment-fragment` for `CommentFragmentComponent` after `CommentFragment`), and extending `ModelEditorComponentBase<T>` where `T` is the fragment's type:
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
import { Fragment, ThesaurusEntry, deepCopy } from '@myrmidon/cadmus-core';
import { AuthService } from '@myrmidon/cadmus-api';
import { ModelEditorComponentBase, DialogService } from '@myrmidon/cadmus-ui';
import { __NAME__Fragment } from '../__NAME__-fragment';

/**
 * __NAME__ fragment editor component.
 * Thesauri: TODO...
 */
@Component({
  selector: 'cadmus-__NAME__-fragment',
  templateUrl: './__NAME__-fragment.component.html',
  styleUrls: ['./__NAME__-fragment.component.css']
})
export class __NAME__FragmentComponent
  extends ModelEditorComponentBase<__NAME__Fragment>
  implements OnInit {
  public tag: FormControl;
  public tags: FormControl;
  public text: FormControl;

  // TODO: add tag entries if required, e.g.:
  // public tagEntries: ThesaurusEntry[];

  constructor(authService: AuthService, formBuilder: FormBuilder) {
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
      this.form.reset();
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
    //   this.tagEntries = null;
    // }
  }

  protected getModelFromForm(): __NAME__Fragment {
    return {
      location: this.model?.location ?? '',
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
      <mat-card-title
        >LingTags Fragment {{ model?.location }}</mat-card-title
      >
      <mat-card-subtitle>
        {{ model?.baseText }}
      </mat-card-subtitle>
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

Remember to add the component to its module's `declarations` and `exports`.

## Adding Fragment to the PG Library

In a `<partgroup>-pg` module:

1. add a _fragment editor feature component_ named after the part (e.g. `ng g component comment-fragment-feature` for `CommentFragmentFeatureComponent` after `CommentFragment`), with routing. Ensure that this component is both under the module `declarations` and in `public-api.ts`. Each editor has its component, and its state management artifacts under the same folder (store, query, and service). Add the corresponding route in the module file, under the `RouterModuleForChild` constant (which is among the `imports` of the module) e.g.:

```ts
export const RouterModuleForChild = RouterModule.forChild([
  // other routes...
  {
    path: `fragment/:pid/${__NAME___FRAGMENT_TYPEID}/:loc`,
    pathMatch: 'full',
    component: __NAME__FragmentFeatureComponent,
    canDeactivate: [PendingChangesGuard]
  }
]);
```

2. inside this new component's folder, add a new **store** for your model, named `edit-<fragmentname>-fragment.store.ts`. Template:

```ts
import { Injectable } from '@angular/core';
import { StoreConfig, Store } from '@datorama/akita';
import {
  EditFragmentState,
  EditFragmentStoreApi,
  editFragmentInitialState
} from '@myrmidon/cadmus-state';
// change this import as required
import { __NAME___FRAGMENT_TYPEID } from '@myrmidon/cadmus-tgr-part-gr-ui';

@Injectable({ providedIn: 'root' })
@StoreConfig({ name: __NAME___FRAGMENT_TYPEID })
export class Edit__NAME__FragmentStore extends Store<EditFragmentState>
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

3. in the same folder, add a new **query** for your model, named `edit-<fragmentname>-fragment.query.ts`. Template:

```ts
import { Injectable } from '@angular/core';
import { EditFragmentQueryBase } from '@myrmidon/cadmus-state';
import { Edit__NAME__FragmentStore } from './edit-__NAME__-fragment.store';

@Injectable({ providedIn: 'root' })
export class Edit__NAME__FragmentQuery extends EditFragmentQueryBase {
  constructor(
    protected store: Edit__NAME__FragmentStore
  ) {
    super(store);
  }
}
```

4. in the same folder, add a new **service** for your model, named `edit-<fragmentname>-fragment.service.ts`. Template:

```ts
import { Injectable } from '@angular/core';
import { ItemService, ThesaurusService } from '@myrmidon/cadmus-api';
import { EditFragmentServiceBase } from '@myrmidon/cadmus-state';
import { Edit__NAME__FragmentStore } from './edit-__NAME__-fragment.store';

@Injectable({ providedIn: 'root' })
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

5. implement the feature editor component by making it extend `EditFragmentFeatureBase`, like in this code template:

```ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import { Edit__NAME__FragmentQuery } from './edit-__NAME__-fragment.query';
import { Edit__NAME__FragmentService } from './edit-__NAME__-fragment.service';
import {
  EditItemQuery,
  EditItemService,
  EditLayerPartQuery,
  EditLayerPartService,
  EditFragmentFeatureBase
} from '@myrmidon/cadmus-state';
import { LibraryRouteService } from '@myrmidon/cadmus-core';

@Component({
  selector: 'cadmus-__NAME__-fragment-feature',
  templateUrl: './__NAME__-fragment-feature.component.html',
  styleUrls: ['./__NAME__-fragment-feature.component.css']
})
export class __NAME__FragmentFeatureComponent extends EditFragmentFeatureBase
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
    this.initEditor(['comment-tags']);
  }
}
```

Define the corresponding HTML template like:

```html
<cadmus-current-item-bar></cadmus-current-item-bar>
<div class="base-text">
  <cadmus-decorated-token-text
    [baseText]="baseText$ | async"
    [locations]="[frLoc]"
  ></cadmus-decorated-token-text>
</div>
<cadmus-__NAME__-fragment
  [model]="fragment$ | async"
  (modelChange)="save($event)"
  [thesauri]="thesauri$ | async"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
></cadmus-__NAME__-fragment>
```

6. in your app's project `part-editor-keys.ts`, add the mapping for the fragment just created under the layer part key, like e.g.:

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
