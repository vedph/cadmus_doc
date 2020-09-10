# Adding Parts

Server-side parts in the editor are essentially used for indexing. Anyway, their model and logic can be used by any other 3rd party processor for its own purposes.

## Parts

Guidelines for **implementing a part**:

- *derive* from `PartBase`, even if this is not strictly a requirement, but rather a commodity. The part class must anyway implement the `IPart` interface.

- *decorate* the class with a `TagAttribute` providing the part's type ID.

- *do not add any logic* to the part. The part is just a POCO object modeling the data it represents, and should have no logic. The only piece of logic required is the method returning the part's data pins, which is just a form of reflecting on the part's data themselves, e.g. for indexing.

- if creating a part representing a *base text* for text layers, implement the `IHasText` interface by providing a `GetText()` method which, whatever the part's model, produces a single string representing its whole text. The same interface should be implemented whenever your part has some rather long piece of free, unstructured text you might want to be included in processes like full-text indexing. Also, set the role ID to `base-text` (defined in `PartBase.BASE_TEXT_ROLE_ID`).

- consider that the part will be subject to automatic serialization and deserialization. As the part is just a POCO object, this should not pose any issue.

**Part template** sample (in the following template replace `__NAME__` with your part's name, minus the `Part` suffix):

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Config;

// ...

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>net.fusisoft.__NAME__</c>.</para>
/// </summary>
[Tag("net.fusisoft.__NAME__")]
public class __NAME__Part : PartBase
{
    // TODO: add your properties here...

    /// <summary>
    /// Get all the key=value pairs (pins) exposed by the implementor.
    /// </summary>
    /// <param name="item">The optional item. The item with its parts
    /// can optionally be passed to this method for those parts requiring
    /// to access further data.</param>
    /// <returns>The pins.</returns>
    public override IEnumerable<DataPin> GetDataPins(IItem item)
    {
        throw new NotImplementedException();

        // TODO: implement indexing logic...
        // sample:
        // return Tag != null
        //    ? new[]
        //    {
        //        CreateDataPin("tag", Tag)
        //    }
        //    : Enumerable.Empty<DataPin>();
    }

    /// <summary>
    /// Converts to string.
    /// </summary>
    /// <returns>
    /// A <see cref="string" /> that represents this instance.
    /// </returns>
    public override string ToString()
    {
        StringBuilder sb = new StringBuilder();

        sb.Append("[__NAME__]");

        // TODO: append summary data...

        return sb.ToString();
    }
}
```

As this is a typical scenario, here is a variant for those parts which essentially just include a list of objects (of type `__ENTRY__`):

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Config;

// ...

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>net.fusisoft.__NAME__</c>.</para>
/// </summary>
[Tag("net.fusisoft.__NAME__")]
public sealed class __NAME__Part : PartBase
{
    /// <summary>
    /// Gets or sets the entries.
    /// </summary>
    public List<__ENTRY__> Entries { get; set; }

    /// <summary>
    /// Initializes a new instance of the <see cref="__NAME__Part"/> class.
    /// </summary>
    public __NAME__Part()
    {
        Entries = new List<__ENTRY__>();
    }

    /// <summary>
    /// Get all the key=value pairs (pins) exposed by the implementor.
    /// </summary>
    /// <param name="item">The optional item. The item with its parts
    /// can optionally be passed to this method for those parts requiring
    /// to access further data.</param>
    /// <returns>The pins: <c>tot-count</c> and a collection of pins with
    /// these keys: ....</returns>
    public override IEnumerable<DataPin> GetDataPins(IItem item)
    {
        DataPinBuilder builder = new DataPinBuilder();

        builder.Set("tot", Entries?.Count ?? 0, false);

        if (Entries?.Count > 0)
        {
            foreach (var entry in Entries)
            {
                // TODO: add values or increase counts like:
                // id unique values if not null:
                // builder.AddValue("id", entry.Id);
                // type-X-count counts if not null, unfiltered:
                // builder.Increase(entry.Type, false, "type-");
            }
        }

        return builder.Build(this);
    }

    /// <summary>
    /// Converts to string.
    /// </summary>
    /// <returns>
    /// A <see cref="string" /> that represents this instance.
    /// </returns>
    public override string ToString()
    {
        StringBuilder sb = new StringBuilder();

        sb.Append("[__NAME__]");

        if (Entries?.Count > 0)
        {
            sb.Append(' ');
            int n = 0;
            foreach (var entry in Entries)
            {
                if (++n > 3) break;
                if (n > 1) sb.Append("; ");
                sb.Append(entry);
            }
            if (Entries.Count > 3)
                sb.Append("...(").Append(Entries.Count).Append(')');
        }

        return sb.ToString();
    }
}
```

**Test template** sample: the part being usually a DTO object, its logic is found only in indexing. Thus, the tests must ensure that the object is serializable, and that the pins are created as expected. In the following template replace `__NAME__` with your part's name, minus the `Part` suffix.

```cs
public sealed class __NAME__PartTest
{
    private static __NAME__Part GetPart()
    {
        return new __NAME__Part
        {
            ItemId = Guid.NewGuid().ToString(),
            RoleId = "some-role",
            CreatorId = "zeus",
            UserId = "another",
            // TODO: set parts data here...
        };
    }

    [Fact]
    public void Part_Is_Serializable()
    {
        __NAME__Part part = GetPart();

        string json = TestHelper.SerializePart(part);
        __NAME__Part part2 = TestHelper.DeserializePart<__NAME__Part>(json);

        Assert.Equal(part.Id, part2.Id);
        Assert.Equal(part.TypeId, part2.TypeId);
        Assert.Equal(part.ItemId, part2.ItemId);
        Assert.Equal(part.RoleId, part2.RoleId);
        Assert.Equal(part.CreatorId, part2.CreatorId);
        Assert.Equal(part.UserId, part2.UserId);
        // TODO: check parts data here...
    }

    // TODO: check pins here, e.g. for the NotePart we get a single pin
    // when the tag is set, with name=tag and value=tag value:
    // [Fact]
    // public void GetDataPins_NoTag_Empty()
    // {
    //     __NAME__Part part = GetPart();
    //     part.Tag = null;

    //     Assert.Empty(part.GetDataPins());
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     __NAME__Part part = GetPart();

    //     List<DataPin> pins = part.GetDataPins().ToList();
    //     Assert.Single(pins);

    //     DataPin pin = pins[0];
    //     Assert.Equal(part.ItemId, pin.ItemId);
    //     Assert.Equal(part.Id, pin.PartId);
    //     Assert.Equal(part.RoleId, pin.RoleId);
    //     Assert.Equal("tag", pin.Name);
    //     Assert.Equal("some-tag", pin.Value);
    // }
}
```

The corresponding test for the list-only part scenario:

```cs
using Cadmus.Core;
using System;
using System.Collections.Generic;
using System.Linq;
using Xunit;

// ...

public sealed class __NAME__PartTest
{
    private static __NAME__Part GetPart(int count)
    {
        __NAME__Part part = new __NAME__Part
        {
            ItemId = Guid.NewGuid().ToString(),
            RoleId = "some-role",
            CreatorId = "zeus",
            UserId = "another",
        };

        for (int n = 1; n <= count; n++)
        {
            part.Entries.Add(new __ENTRY__
            {
                // TODO: set properties
            });
        }

        return part;
    }

    [Fact]
    public void Part_Is_Serializable()
    {
        __NAME__Part part = GetPart(2);

        string json = TestHelper.SerializePart(part);
        __NAME__Part part2 =
            TestHelper.DeserializePart<__NAME__Part>(json);

        Assert.Equal(part.Id, part2.Id);
        Assert.Equal(part.TypeId, part2.TypeId);
        Assert.Equal(part.ItemId, part2.ItemId);
        Assert.Equal(part.RoleId, part2.RoleId);
        Assert.Equal(part.CreatorId, part2.CreatorId);
        Assert.Equal(part.UserId, part2.UserId);

        Assert.Equal(2, part.Entries.Count);
        // TODO: details
    }

    [Fact]
    public void GetDataPins_NoEntries_Ok()
    {
        __NAME__Part part = GetPart(0);

        List<DataPin> pins = part.GetDataPins(null).ToList();

        Assert.Single(pins);
        DataPin pin = pins[0];
        Assert.Equal("tot-count", pin.Name);
        TestHelper.AssertPinIds(part, pin);
        Assert.Equal("0", pin.Value);
    }

    [Fact]
    public void GetDataPins_Entries_Ok()
    {
        __NAME__Part part = GetPart(3);

        List<DataPin> pins = part.GetDataPins(null).ToList();

        Assert.Equal(5, pins.Count);

        DataPin pin = pins.Find(p => p.Name == "tot-count");
        Assert.NotNull(pin);
        TestHelper.AssertPinIds(part, pin);
        Assert.Equal("3", pin.Value);

        // TODO: assert counts and values e.g.:
        // pin = pins.Find(p => p.Name == "pos-bottom-count");
        // Assert.NotNull(pin);
        // TestHelper.AssertPinIds(part, pin);
        // Assert.Equal("2", pin.Value);
    }
}
```

## Layer Parts

For layer parts, the same guidelines already listed for the other parts are applicable, with the following additions:

- create a `...LayerFragment` class representing the fragment for the layer part. This is the true data model for the metatextual data represented by the layer. The class must implement `ITextLayerFragment`. Do not add any other property to the class; *by design, the only property of a layer part is its collection of fragments*.

- give the fragment a type ID (via the usual `TagAttribute`), which *must* begin with the prefix `fr.` (=`PartBase.FR_PREFIX`; note the trailing dot).

- if adding pins in the fragment, just provide the pin's name and value; the other properties will be supplied by the container part. By convention, you should prefix your pin name with the `fr.` prefix (defined in `PartBase.FR_PREFIX`).

Anyway, adding a new layer part would be rarely required, as there is just a generic (parameterized) layer part provided for this: one part, many fragments. You rather have to provide fragments and their tests.

**Fragment test template** sample:

```cs
public sealed class __NAME__LayerFragmentTest
{
    private static __NAME__LayerFragment GetFragment()
    {
        return new __NAME__LayerFragment
        {
            Location = "1.23",
            // TODO: add properties here...
        };
    }

    [Fact]
    public void Fragment_Has_Tag()
    {
        TagAttribute attr = typeof(__NAME__LayerFragment).GetTypeInfo()
            .GetCustomAttribute<TagAttribute>();
        string typeId = attr != null ? attr.Tag : GetType().FullName;
        Assert.NotNull(typeId);
        Assert.StartsWith(PartBase.FR_PREFIX, typeId);
    }

    [Fact]
    public void Fragment_Is_Serializable()
    {
        __NAME__LayerFragment fr = GetFragment();

        string json = TestHelper.SerializeFragment(fr);
        __NAME__LayerFragment fr2 =
            TestHelper.DeserializeFragment<__NAME__LayerFragment>(json);

        Assert.Equal(fr.Location, fr2.Location);
        // TODO: check properties here...
    }

    // TODO: check pins here, e.g. for the CommentLayerFragment
    // we get a single pin when the tag is set, with name=fr.tag
    // and value=tag value:
    // [Fact]
    // public void GetDataPins_NoTag_0()
    // {
    //     CommentLayerFragment fr = GetFragment();
    //     fr.Tag = null;

    //     Assert.Empty(fr.GetDataPins());
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     CommentLayerFragment fr = GetFragment();

    //     List<DataPin> pins = fr.GetDataPins().ToList();

    //     Assert.Single(pins);
    //     DataPin pin = pins[0];
    //     Assert.Equal("fr.tag", pin.Name);
    //     Assert.Equal("some-tag", pin.Value);
    // }
}
```
