# Editing Item

When editing an item, you have 4 sections dedicated to metadata, parts, layers, and thesaurus scopes.

Whatever section is selected, you always have at the top of the screen the item's title and description, its ID, and the last saved and created date and time.

**►TIP**: you can easily copy the item's ID by just clicking on it. Items and part IDs can be used to directly work with the underlying database, as they identify the corresponding documents in the MongoDB database.

## Metadata

The metadata section allows you to edit the item's metadata:

- **title**: a (mostly arbitrary) title for the item. This is used as the base for sorting items in browsing them.
- **description**: a short description about the item.
- **facet**: the facet the item belongs to. You can select the facet from the list of available facets, which is derived from the Cadmus profile.
- **group ID**: an arbitrary ID used to group any number of items together. The exact meaning of the ID varies according to the data; for instance, in a set of text-based items representing each some number of verses from the *Iliad*, these items could be grouped by their book number. In this example, all the items belonging to book 1 could have their group ID equal to `01`; all the items belonging to book 2 could have their group ID equal to `02`; etc. To remove the grouping just assign a blank group ID.
- **flags**: item's flags with their labels and colors are defined in the Cadmus profile. When flags are defined, they appear listed at the bottom of the section. You can check or uncheck each flag as desired.

Other data like ID, sort ID, or timestamps are displayed but not editable, because they are managed by the system.

Once you have edited any of these metadata, you must save them by clicking the save button.

## Parts

The parts section lists all the item's parts. Each part is listed with 3 buttons for editing or deleting it, or copying its ID.

To **add a new non-layer part** (for layer parts, see the [Layers section](#layers)), just select its type from the dropdown list, and click the `New part` button. The list contains all the available non-layer parts, eventually combined with their role IDs when specified. Thus, if you have a generic note part and a note part with role ID equal to "scholarly", then two different note parts will be listed.

When **editing a part**, you are directed to the specialized editor for its type.

When **deleting a part**, you are prompted for confirmation; after that, the part will be directly removed.

## Layers

The layers section lists only the layer parts, allowing you to edit, delete or copy the ID of each existing part, or to add a new layer part among those defined in the Cadmus profile and not yet added to the current item.

This is a list of all the *available* layer parts; some may be existing, and others are not. For existing parts, you also get the count of their fragments, and their last modification date and time.

## Th-Scopes

The thesauri scopes section lists all the existing item's parts, and allows you to assign a specific [thesaurus scope](../core/../../core/profiles.md#thesaurus-scope-in-parts) to all the checked parts in this list. To do this, just toggle the checkmark for all the parts you want to assign the scope ID to; type the scope ID; and click the `assign` button.

**►INFO** A *thesaurus scope* allows to pick a different thesaurus according to the scope ID stored in the part. The ID is the final part of a thesaurus ID, following the last dot and without the language suffix: e.g. in `apparatus-witnesses.verg-eclo` the scope ID is `verg-eclo`. All the parts with this scope ID assigned will instruct the corresponding editor to pick the thesaurus specialized for that scope, so that we can change the lookup data of that editor at runtime.
