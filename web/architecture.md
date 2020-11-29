# Web App Architecture

The general web app code architecture is structured into the following sections.

## Application

- `cadmus-shell/`: the frontend app, used only for demo and development purposes. Each frontend is then independently built following its model.

## Libraries

Each library is a project under `cadmus-shell/projects/myrmidon`, and gets published in NPM. Each Cadmus web app then just imports the libraries it requires for its operations.

## Editing Architecture

From the standpoint of the editing process, the architecture is hierarchical: we start from a list of items, edit an item from it, edit a part from that item, and eventually edit a fragment from that part (for layer parts).

### Items List

- store: `ItemsStore` including `ItemsState` and extending `EntityState<ItemInfo, string>`. The only edit operation here is deleting an item, via `ItemsListService`, which invokes the deletion API and removes the deleted item from the store.
- component: `ItemListComponent`.

Operations:

- *add item*: move to item editor (`/items/new`).
- *edit item*: move to item editor (`/items/<id>`).
- *delete item*: delete via `ItemsListService`.

### Add or Edit Item

- store: `EditItemStore` including `EditItemState` and extending `Store<EditItemState>`. The service handling this store is `EditItemService`, which can load an item, save an item, delete an item's part, add a new layer part, and set a part's thesaurus scope.
- component: `ItemEditorComponent`.

Operations:

- *save metadata*: via `EditItemService`.
- *add/edit a non-layer part*: move to part editor (`/items/<id>/<part-group>/<part-typeid>/<part-id>?rid=<role-id>`).
- *delete a part*: delete via `EditItemService`.
- *add a layer part*: via `EditItemService`. The part gets added with no fragments. You will then edit it.
- *edit a layer part*: move to layer part editor (same route as above for editing non-layer parts).
- *set thesauri scopes* for selected parts: via `EditItemService`.

### Edit Part

Editing a part implies a lot of different components, organized in a structure which is common to all the parts.

This structure is fully wrapped in two libraries which provide part and fragment editors to the shell. The first library (known as ui-library) contains the core editor components; the second library (known as pg-library) depends on the first, and it is just a wrapper for the ui-library components, connecting them to the shell application.

Part editors are all found in lazily-loaded libraries, each at its own route like `/items/<id>/<part-group>/<part-typeid>/<part-id>?rid=<role-id>` (where `rid` is optional). These routes are provided by the pg-library.

Fragment editors, like part editors, are all found in lazily loaded libraries, each at its own route like `/items/<id>/<part-group>/fragment/<part-id>/<fr-typeid>/<loc>?rid=<role-id>` (where `rid` is optional).

#### Part Editor Component

At the base we have a *part editor component*, which is the UI in charge of editing the specific part's data model.

By convention, a part editor UI is named like `<PartName>PartComponent`.

This is a component with any UI, extending the base class `ModelEditorComponentBase<T>`. Even if deriving your component from this class is not a requirement, it's the suggested way of building editors, because it provides the wiring between the component UI and its common functions (like loading and saving model data, loading thesauri, and initialize common resources):

- `initEditor` must be called in `ngOnInit`.
- `onModelSet(model: T)` is handled to update the form from the received part.
- `onThesauriSet()` is handled to get taxonomies from thesauri once the required thesauri have been loaded.
- `getModelFromForm(): T` is called when the user wants to save the edited part. It does the inverse of `onModelSet`, building the part model from the form.

The part editor exposes this interface (via its base class):

- input:
  - item ID
  - role ID
  - the part
  - thesauri
  - disabled
- output:
  - part change
  - editor close request
  - dirty state change

#### Part Feature Component

This is the component of the pg-library, wrapping the corresponding `<PartName>EditorComponent`. In turn, it extends `EditPartFeatureBase`, which connects the wrapped component to an edit route.

The base class extracts item ID, part ID (equal to `new` if the part is new), and role ID (equal to `default` for no specific role) of the edited part from the route which lead to itself.

Also, the base class has these **dependencies**:

- the Akita **part query** component of the part being edited.
- the Akita **part service** component of the part being edited.
- the Akita **item query** component of the item including the part being edited.
- the Akita **item service** component of the item including the part being edited.

The part being edited and its thesauri are exposed via two observables (`part$` and `thesauri$`). These are connected to the Akita part **query**. The part query is used to fetch the part (and its thesauri) from its store. Each part query extends `EditPartQueryBase`, and receives the Akita store for its part.

In turn, the **store** implements the interface `EditPartStoreApi`, and extends Akita `Store<EditPartState>`. `EditPartState` represents the state of the part being edited, including:

- the part
- thesauri
- dirty flag
- saving flag
- loading flag
- eventual error message

The base class implements `ComponentCanDeactivate` to allow using the `PendingChangesGuard`. To this end, it tracks the dirty state of the edited data coming from 2 different sources:

- the wrapped editor state (from its "root" form).
- the store edit state. This is set to dirty when a save attempt fails.

Thus, at the end the user will be prompted when closing an editor either because he has changed data in it, or because he attempted a save without success.

### Edit Fragment

Editing fragments is a special case of part editing. In fact, when loading and saving data, it's the whole layer part which gets transferred. Yet, the part being edited is just a collection of fragments. Each fragment gets edited in its own UI, using the corresponding fragment editor.

#### Fragment Editor Component

A fragment is edited in the UI provided by the *fragment editor component*, conventionally named like `<FragmentName>FragmentComponent`.

As seen above for the part, it extends the same `ModelEditorComponentBase<T>` class.

#### Fragment Feature Component

The fragment editor component is wrapped in an editor in the pg-library, named like `<FragmentName>FragmentFeatureComponent`.

In turn, it extends `EditFragmentFeatureBase`, which connects the wrapped component to an edit route.

The base class extracts from its route the item ID, part ID (equal to `new` if the part is new), role ID (equal to `default` for no specific role), of the edited part; and the fragment type ID, location and role ID.

Also, the base class has these **dependencies**:

- the Akita **fragment query** component of the fragment being edited.
- the Akita **fragment service** component of the fragment being edited.
- the Akita **item query** component of the item including the part being edited.
- the Akita **item service** component of the item including the part being edited.
- the Akita **layers query** component.
- the Akita **layers service** component.

The JSON code of the part being edited and its thesauri are exposed via two observables (`json$` and `thesauri$`); the base text of the fragment is exposed in another observable (`baseText$`). These are connected to the Akita fragment and layers **query**. The fragment query is used to fetch the fragment (and its thesauri) from its store. Each fragment query extends `EditFragmentQueryBase`, and receives the Akita store for its fragment.

In turn, the **store** implements the interface `EditFragmentStoreApi`, and extends Akita `Store<EditFragmentState>`. `EditFragmentState` represents the state of the fragment being edited, including:

- the fragment
- thesauri
- dirty flag
- saving flag
- loading flag
- eventual error message

The base class implements `ComponentCanDeactivate` to allow using the `PendingChangesGuard`. To this end, it tracks the dirty state of the edited data coming from 2 different sources:

- the wrapped editor state (from its "root" form).
- the store edit state. This is set to dirty when a save attempt fails.

Thus, at the end the user will be prompted when closing an editor either because he has changed data in it, or because he attempted a save without success.
