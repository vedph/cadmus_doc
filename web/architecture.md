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

- store: `Edit<PartName>PartStore` extending `Store<EditPartState>`. The store ID is equal to the part type ID. The service `Edit<PartName>PartService` extends `EditPartServiceBase`; it uses an `ItemService` and a `ThesaurusService` to exchange data with the backend, and manages its `Edit<PartName>PartStore` store. The query `Edit<PartName>PartQuery` extends `EditPartQueryBase`, in turn extending `Query<EditPartState>`.
- component: `<PartName>PartComponent`, `<PartName>PartFeatureComponent`.

Part editors are all found in lazily-loaded libraries, each at its own route like `/items/<id>/<part-group>/<part-typeid>/<part-id>?rid=<role-id>` (where `rid` is optional).

Each part editor is a dumb UI component extending `ModelEditorComponentBase<ModelType>`; it gets and emits the edited model serialized in a JSON string. So, its implementation consists of its own editing logic and UI, plus some plumbing for updating the UI from the model, getting the model from the UI, and getting thesauri when required.

Each of these editor components is wrapped inside a part editor feature component, which extends `EditPartFeatureBase` and corresponds to a route in the application. Its UI contains only the current item bar at the top, and the dumb editor component.

### Edit Fragment

- store: `Edit<FragmentName>FragmentStore` extending `Store<EditFragmentState>`. `Edit<FragmentName>FragmentService` extends `EditFragmentServiceBase`. `Edit<FragmentName>FragmentQuery` extends `EditFragmentQueryBase`.
- component: `<FragmentName>FragmentComponent` extending `ModelEditorComponentBase<ModelType>`, `<FragmentName>FragmentFeatureComponent` extending `EditFragmentFeatureBase`.

Fragment editors, like part editors, are all found in lazily loaded libraries, each at its own route like `/items/<id>/<part-group>/fragment/<part-id>/<fr-typeid>/<loc>?rid=<role-id>` (where `rid` is optional).

Each fragment editor is a dumb UI component extending `ModelEditorComponentBase<ModelType>`; it gets and emits the edited model serialized in a JSON string. So, its implementation consists of its own editing logic and UI, plus some plumbing for updating the UI from the model, getting the model from the UI, and getting thesauri when required.

Each of these editor components is wrapped inside a fragment editor feature component, which extends `EditFragmentFeatureBase`, and corresponds to a route in the application. Its UI contains only the current item bar, the decorated base text, and the dumb editor component.
