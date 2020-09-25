# Searching Items

This pane allows searching for items via their metadata and their parts data pins.

The search is based on a very simple query language, based on name/value pairs connected by an operator.

Pairs are included between `[]`. Each pair has three components: name, operator + field value. You can specify one or more pairs, connecting them with `AND` and `OR`, and group them using brackets. For instance, this query:

```txt
[title*=test] AND ([facet=default] OR [facet=tiled])
```

finds all the items whose title includes the text `test` and whose facet ID is either `default` or `tiled`.

The pair name can be any of the following:

- `t`, `title`: item's title.
- `dsc`, `description`: item's description.
- `facet`, `facetId`: item's facet ID.
- `group`, `groupId`: item's group ID.
- `sortKey`: item's sort key.
- `flags`: item's flags.
- `type`, `partType`, `partTypeId`: part type ID.
- `role`, `roleId`: part role ID.
- `n`, `name`: pin's name.
- `v`, `value`: pin's value.

Pins names depend on the parts used and which pins are emitted by each of them.

You can also use escapes in the value, by prefixing its 16-bits Unicode hex code value with a backslash. Also, you can insert square brackets by prefixing them with a backslash.

The operators are:

- `=`: equal to.
- `<>`: not equal to.
- `*=`: contains.
- `^=`: starts with.
- `$=`: ends with.
- `?=`: wildcards: `?`=any single character, `*`=0-N characters.
- `~=`: regular expression.
- `%=`: fuzzy matching. The value is followed by `:` and the minimum similarity treshold.
- `==`: equal (numeric).
- `!=`: not equal (numeric).
- `<`: less-than (numeric).
- `>`: greater-than (numeric).
- `<=`: less-than or equal (numeric).
- `>=`: greater-than or equal (numeric).
- `:` (flags): any the specified flags (comma-separated) must be present.
- `&:` (flags): all the specified flags (comma-separated) must be present.
- `!:` (flags): none of the specified flags (comma-separated) must be present.

Flags can be specified either with name or value.
