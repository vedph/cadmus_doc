# Adding Parts

- [Adding Parts](#adding-parts)
  - [Parts](#parts)
    - [Procedure](#procedure)
    - [Part - Single Entity](#part---single-entity)
    - [Part - Multiple Entities](#part---multiple-entities)
    - [Part Seeder](#part-seeder)
    - [Part Seeder Test](#part-seeder-test)
      - [Test Helper](#test-helper)
    - [Part Test - Single Entity Using Seeder](#part-test---single-entity-using-seeder)
    - [Part Test - Multiple Entities Using Seeder](#part-test---multiple-entities-using-seeder)
    - [Part Test - Single Entity Without Seeder](#part-test---single-entity-without-seeder)
    - [Part Test - Multiple Entities Without Seeder](#part-test---multiple-entities-without-seeder)
    - [Test Helper for Parts](#test-helper-for-parts)
  - [Layer Parts](#layer-parts)
    - [Layer Fragment Test Template](#layer-fragment-test-template)

Server-side parts in the editor are essentially used for indexing. Anyway, their model and logic can be used by any other 3rd party processor for its own purposes.

## Parts

Guidelines for **implementing a part**:

- _derive_ from `PartBase`, even if this is not strictly a requirement, but rather a commodity. The part class must anyway implement the `IPart` interface.

- _decorate_ the class with a `TagAttribute` providing the part's type ID.

- _do not add any logic_ to the part. The part is just a POCO object modeling the data it represents, and should have no logic. The only piece of logic required is the method returning the part's data pins, which is just a form of reflecting on the part's data themselves, e.g. for indexing or semantic projection.

- if creating a part representing a _base text_ for text layers, implement the `IHasText` interface by providing a `GetText()` method which, whatever the part's model, produces a single string representing its whole text. The same interface should be implemented whenever your part has some rather long piece of free, unstructured text you might want to be included in processes like full-text indexing. Also, when the part represents the base text in a stack of textual layers, set the role ID to `base-text` (defined in `PartBase.BASE_TEXT_ROLE_ID`).

- consider that the part will be subject to automatic serialization and deserialization. As the part is just a POCO object, this should not pose any issue.

### Procedure

The typical procedure when adding a new part is:

1. create the part class.
2. create the part seeder class.
3. create the part seeder test.
4. create the part test (using both the part under test and its part seeder).

### Part - Single Entity

In the following template replace `__NAME__` with your part's name, minus the `Part` suffix:

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Config;

// ...

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>it.vedph.__PRJ__.__NAME__</c>.</para>
/// </summary>
[Tag("it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__Part : PartBase
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
        // TODO: build pins, eventually using DataPinBuilder like this:
        //DataPinBuilder builder = new DataPinBuilder(
        //    new StandardDataPinTextFilter());

        //// tot-count
        //builder.Set("tot", Entries?.Count ?? 0, false);

        //return builder.Build(this);

        // ...or just use a simpler logic, like:
        // sample:
        // return Tag != null
        //    ? new[]
        //    {
        //        CreateDataPin("tag", Tag)
        //    }
        //    : Enumerable.Empty<DataPin>();

        throw new NotImplementedException();
    }

    /// <summary>
    /// Gets the definitions of data pins used by the implementor.
    /// </summary>
    /// <returns>Data pins definitions.</returns>
    public override IList<DataPinDefinition> GetDataPinDefinitions()
    {
        return new List<DataPinDefinition>(new[]
        {
            // TODO: add pins definitions...
            // sample:
            // new DataPinDefinition(DataPinValueType.Integer,
            //    "tot-count",
            //    "The total count of entries.")
        });
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

### Part - Multiple Entities

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
/// <para>Tag: <c>it.vedph.__PRJ__.__NAME__</c>.</para>
/// </summary>
[Tag("it.vedph.__PRJ__.__NAME__")]
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
    /// Gets the definitions of data pins used by the implementor.
    /// </summary>
    /// <returns>Data pins definitions.</returns>
    public override IList<DataPinDefinition> GetDataPinDefinitions()
    {
        return new List<DataPinDefinition>(new[]
        {
            // TODO: add pins definitions...
            new DataPinDefinition(DataPinValueType.Integer,
               "tot-count",
               "The total count of entries.")
        });
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

### Part Seeder

Add a `<NAME>PartSeeder.cs` for the seeder (replace `__NAME__` with the part name, using the proper case, and adjust the namespace).

If the seeder does not require configuration options, remove the `__NAME__PartSeederOptions` class, the corresponding `_options` member, and the `IConfigurable<T>` interface.

```cs
using Bogus;
using Cadmus.Core;
using Fusi.Tools.Config;
using System;

// ...

/// <summary>
/// Seeder for __NAME__ part.
/// Tag: <c>seed.it.vedph.__PRJ__.__NAME__</c>.
/// </summary>
/// <seealso cref="Cadmus.Seed.PartSeederBase" />
[Tag("seed.it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__PartSeeder : PartSeederBase,
    IConfigurable<__NAME__PartSeederOptions>
{
    private __NAME__PartSeederOptions _options;

    /// <summary>
    /// Configures the object with the specified options.
    /// </summary>
    /// <param name="options">The options.</param>
    public void Configure(__NAME__PartSeederOptions options)
    {
        _options = options;
    }

    /// <summary>
    /// Creates and seeds a new part.
    /// </summary>
    /// <param name="item">The item this part should belong to.</param>
    /// <param name="roleId">The optional part role ID.</param>
    /// <param name="factory">The part seeder factory. This is used
    /// for layer parts, which need to seed a set of fragments.</param>
    /// <returns>A new part.</returns>
    /// <exception cref="ArgumentNullException">item or factory</exception>
    public override IPart GetPart(IItem item, string roleId,
        PartSeederFactory factory)
    {
        if (item == null)
            throw new ArgumentNullException(nameof(item));
        // for layer parts only:
        // if (factory == null)
        //    throw new ArgumentNullException(nameof(factory));

        // TODO: add more options validation check; if invalid, ret null
        if (_options == null)
        {
            return null;
        }

        __NAME__Part part = new __NAME__Part();
        // or with Bogus:
        // __NAME__Part part = new Faker<__NAME__Part>()
        //    .RuleFor(p => p.X, f => TODO)
        //    .Generate();
        SetPartMetadata(part, roleId, item);

        // TODO: add seed code here...

        return part;
    }
}

/// <summary>
/// Options for <see cref="__NAME__PartSeeder"/>.
/// </summary>
public sealed class __NAME__PartSeederOptions
{
    // TODO: add options here...
}
```

### Part Seeder Test

Template for a seeder:

```cs
using Cadmus.Core;
using Fusi.Tools.Config;
using System;
using System.Reflection;
using Xunit;

// ...
public sealed class __NAME__PartSeederTest
{
    private static readonly PartSeederFactory _factory;
    private static readonly SeedOptions _seedOptions;
    private static readonly IItem _item;

    static __NAME__PartSeederTest()
    {
        _factory = TestHelper.GetFactory();
        _seedOptions = _factory.GetSeedOptions();
        _item = _factory.GetItemSeeder().GetItem(1, "facet");
    }

    [Fact]
    public void TypeHasTagAttribute()
    {
        Type t = typeof(__NAME__PartSeeder);
        TagAttribute? attr = t.GetTypeInfo().GetCustomAttribute<TagAttribute>();
        Assert.NotNull(attr);
        Assert.Equal("seed.it.vedph.__PRJ__.__NAME__", attr!.Tag);
    }

    [Fact]
    public void Seed_Ok()
    {
        __NAME__PartSeeder seeder = new();
        seeder.SetSeedOptions(_seedOptions);

        IPart part = seeder.GetPart(_item, null, _factory);

        Assert.NotNull(part);

        __NAME__Part? p = part as __NAME__Part;
        Assert.NotNull(p);

        TestHelper.AssertPartMetadata(p!);

        // TODO: assert properties like:
        // Assert.NotEmpty(p!.Works);
    }
}
```

#### Test Helper

This template requires some infrastructure files:

- a minimalist JSON configuration file for the seeders to be tested: `SeedConfig.json` (embedded resource) under your test project's `Assets` folder.
- a `TestHelper` to use this configuration.
- package `Fusi.Microsoft.Extensions.Configuration.InMemoryJson` to read the configuration from the embedded `SeedConfig.json`.

Sample configuration:

```json
{
  "facets": [
    {
      "typeId": "it.vedph.pura.word-forms",
      "name": "forms",
      "description": "Word forms.",
      "colorKey": "31AB54",
      "groupKey": "lexicon",
      "sortKey": "forms"
    }
  ],
  "seed": {
    "options": {
      "seed": 1,
      "baseTextPartTypeId": "it.vedph.token-text",
      "users": [ "zeus" ],
      "partRoles": [],
      "fragmentRoles": []
    },
    "partSeeders": [
      {
        "id": "seed.it.vedph.pura.word-forms"
      }
    ],
    "fragmentSeeders": []
  }
}
```

Here we just add all the parts to a single facet, and their seeders to be tested.

Sample `TestHelper`:

```cs
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Pura.Parts;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Microsoft.Extensions.Configuration;
using SimpleInjector;
using System;
using System.IO;
using System.Reflection;
using System.Text;
using Xunit;

// ...

static internal class TestHelper
{
    static public Stream GetResourceStream(string name)
    {
        if (name == null) throw new ArgumentNullException(nameof(name));

        return Assembly.GetExecutingAssembly().GetManifestResourceStream(
                $"Cadmus.Seed.Pura.Parts.Test.Assets.{name}");
    }

    static public string LoadResourceText(string name)
    {
        if (name == null) throw new ArgumentNullException(nameof(name));

        using (StreamReader reader = new StreamReader(
            GetResourceStream(name),
            Encoding.UTF8))
        {
            return reader.ReadToEnd();
        }
    }

    static public PartSeederFactory GetFactory()
    {
        // map
        TagAttributeToTypeMap map = new TagAttributeToTypeMap();
        map.Add(new[]
        {
            // Cadmus.Core
            typeof(StandardItemSortKeyBuilder).Assembly,
            // Cadmus.Philology.Parts
            typeof(ApparatusLayerFragment).Assembly
        });

        // container
        Container container = new Container();
        PartSeederFactory.ConfigureServices(
            container,
            new StandardPartTypeProvider(map),
            new[]
            {
                // Cadmus.Seed.Parts
                typeof(NotePartSeeder).Assembly,
                // Cadmus.Seed.Philology.Parts
                typeof(ApparatusLayerFragmentSeeder).Assembly
            });

        // config
        IConfigurationBuilder builder = new ConfigurationBuilder()
            .AddInMemoryJson(LoadResourceText("SeedConfig.json"));
        var configuration = builder.Build();

        return new PartSeederFactory(container, configuration);
    }

    static public void AssertPartMetadata(IPart part)
    {
        Assert.NotNull(part.Id);
        Assert.NotNull(part.ItemId);
        Assert.NotNull(part.UserId);
        Assert.NotNull(part.CreatorId);
    }
}
```

### Part Test - Single Entity Using Seeder

This template takes advantage of the part's seeder. If you are not using a seeder, refer to the [next section](#part-test-template---no-seeder).

```cs
using System;
using Xunit;
using Cadmus.Core;
using System.Collections.Generic;
using System.Linq;

public sealed class __NAME__PartTest
{
    private static __NAME__Part GetPart()
    {
        __NAME__PartSeeder seeder = new __NAME__PartSeeder();
        IItem item = new Item
        {
            FacetId = "default",
            CreatorId = "zeus",
            UserId = "zeus",
            Description = "Test item",
            Title = "Test Item",
            SortKey = ""
        };
        return (__NAME__Part)seeder.GetPart(item, null, null);
    }

    private static __NAME__Part GetEmptyPart()
    {
        return new __NAME__Part
        {
            ItemId = Guid.NewGuid().ToString(),
            RoleId = "some-role",
            CreatorId = "zeus",
            UserId = "another",
        };
    }

    [Fact]
    public void Part_Is_Serializable()
    {
        __NAME__Part part = GetPart();

        string json = TestHelper.SerializePart(part);
        __NAME__Part part2 = TestHelper.DeserializePart<__NAME__Part>(json)!;

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
    //     __NAME__Part part = GetEmptyPart(null);
    //     part.Tag = null;

    //     Assert.Empty(part.GetDataPins());
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     __NAME__Part part = GetEmptyPart();
    //     // TODO: set only the properties required for pins
    //     // in a predictable way so we can test them

    //     List<DataPin> pins = part.GetDataPins(null).ToList();
    //     Assert.Single(pins);

    //     DataPin pin = pins[0];
    //     Assert.Equal(part.ItemId, pin.ItemId);
    //     Assert.Equal(part.Id, pin.PartId);
    //     Assert.Equal(part.RoleId, pin.RoleId);
    //     Assert.Equal("tag", pin.Name);
    //     Assert.Equal("some-tag", pin.Value);
    //     // another way:
    //     pin = pins.Find(p => p.Name == "id" && p.Value == "steph");
    //     Assert.NotNull(pin);
    //     TestHelper.AssertPinIds(part, pin);
    // }
}
```

### Part Test - Multiple Entities Using Seeder

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
    private static __NAME__Part GetPart()
    {
        __NAME__PartSeeder seeder = new();
        IItem item = new Item
        {
            FacetId = "default",
            CreatorId = "zeus",
            UserId = "zeus",
            Description = "Test item",
            Title = "Test Item",
            SortKey = ""
        };
        return (__NAME__Part)seeder.GetPart(item, null, null);
    }

    private static __NAME__Part GetEmptyPart()
    {
        return new __NAME__Part
        {
            ItemId = Guid.NewGuid().ToString(),
            RoleId = "some-role",
            CreatorId = "zeus",
            UserId = "another",
        };
    }

    [Fact]
    public void Part_Is_Serializable()
    {
        __NAME__Part part = GetPart();

        string json = TestHelper.SerializePart(part);
        __NAME__Part part2 =
            TestHelper.DeserializePart<__NAME__Part>(json)!;

        Assert.Equal(part.Id, part2.Id);
        Assert.Equal(part.TypeId, part2.TypeId);
        Assert.Equal(part.ItemId, part2.ItemId);
        Assert.Equal(part.RoleId, part2.RoleId);
        Assert.Equal(part.CreatorId, part2.CreatorId);
        Assert.Equal(part.UserId, part2.UserId);

        Assert.Equal(part.Entries.Count, part2.Entries.Count);
    }

    [Fact]
    public void GetDataPins_NoEntries_Ok()
    {
        __NAME__Part part = GetPart();
        part.Entries.Clear();

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
        __NAME__Part part = GetEmptyPart();

        for (int n = 1; n <= 3; n++)
        {
            // TODO add entry to part setting its pin-related
            // properties in a predictable way, so we can test them
        }

        List<DataPin> pins = part.GetDataPins(null).ToList();

        Assert.Equal(5, pins.Count);

        DataPin? pin = pins.Find(p => p.Name == "tot-count");
        Assert.NotNull(pin);
        TestHelper.AssertPinIds(part, pin!);
        Assert.Equal("3", pin!.Value);

        // TODO: assert counts and values e.g.:
        // pin = pins.Find(p => p.Name == "pos-bottom-count");
        // Assert.NotNull(pin);
        // TestHelper.AssertPinIds(part, pin!);
        // Assert.Equal("2", pin.Value);
    }
}
```

### Part Test - Single Entity Without Seeder

The part being usually a DTO object, its logic is found only in indexing. Thus, the tests must ensure that the object is serializable, and that the pins are created as expected. In the following template replace `__NAME__` with your part's name, minus the `Part` suffix.

```cs
using System;
using Xunit;
using Cadmus.Core;

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

    //     Assert.Empty(part.GetDataPins(null));
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     __NAME__Part part = GetPart();

    //     List<DataPin> pins = part.GetDataPins(null).ToList();
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

### Part Test - Multiple Entities Without Seeder

The corresponding test for the list-only part scenario:

```cs
using Cadmus.Core;
using System;
using System.Collections.Generic;
using System.Linq;
using Xunit;

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

        Assert.Equal(part.Entries.Count, part2.Entries.Count);
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

### Test Helper for Parts

A typical parts test helper:

```cs
using Cadmus.Core;
using Cadmus.Core.Layers;
using Fusi.Antiquity.Chronology;
using System;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.RegularExpressions;
using Xunit;

// ...

internal sealed class TestHelper
{
    private static readonly JsonSerializerOptions _options =
        new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

    public static string SerializePart(IPart part)
    {
        if (part == null)
            throw new ArgumentNullException(nameof(part));

        return JsonSerializer.Serialize(part, part.GetType(), _options);
    }

    public static T DeserializePart<T>(string json)
        where T : class, IPart, new()
    {
        if (json == null)
            throw new ArgumentNullException(nameof(json));

        return JsonSerializer.Deserialize<T>(json, _options);
    }

    public static string SerializeFragment(ITextLayerFragment fr)
    {
        if (fr == null)
            throw new ArgumentNullException(nameof(fr));

        return JsonSerializer.Serialize(fr, fr.GetType(), _options);
    }

    public static T DeserializeFragment<T>(string json)
        where T : class, ITextLayerFragment, new()
    {
        if (json == null)
            throw new ArgumentNullException(nameof(json));

        return JsonSerializer.Deserialize<T>(json, _options);
    }

    public static void AssertPinIds(IPart part, DataPin pin)
    {
        Assert.Equal(part.ItemId, pin.ItemId);
        Assert.Equal(part.Id, pin.PartId);
        Assert.Equal(part.RoleId, pin.RoleId);
    }

    static public bool IsDataPinNameValid(string name) =>
        Regex.IsMatch(name, @"^[a-zA-Z0-9\-_\.]+$");

    static public void AssertValidDataPinNames(IList<DataPin> pins)
    {
        foreach (DataPin pin in pins)
        {
            Assert.True(IsDataPinNameValid(pin.Name), pin.ToString());
        }
    }
}
```

## Layer Parts

For layer parts, the same guidelines already listed for the other parts are applicable, with the following additions:

- create a `...LayerFragment` class representing the fragment for the layer part. This is the true data model for the metatextual data represented by the layer. The class must implement `ITextLayerFragment`. Do not add any other property to the class; _by design, the only property of a layer part is its collection of fragments_.

- give the fragment a type ID (via the usual `TagAttribute`), which _must_ begin with the prefix `fr.` (=`PartBase.FR_PREFIX`; note the trailing dot).

- if adding pins in the fragment, just provide the pin's name and value; the other properties will be supplied by the container part. By convention, you should prefix your pin name with the `fr.` prefix (defined in `PartBase.FR_PREFIX`).

Anyway, adding a new layer part would be rarely required, as there is just a generic (parameterized) layer part provided for this: one part, many fragments. You rather have to provide fragments and their tests.

### Layer Fragment Test Template

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

    //     Assert.Empty(fr.GetDataPins(null));
    // }

    // [Fact]
    // public void GetDataPins_Tag_1()
    // {
    //     CommentLayerFragment fr = GetFragment();

    //     List<DataPin> pins = fr.GetDataPins(null).ToList();

    //     Assert.Single(pins);
    //     DataPin pin = pins[0];
    //     Assert.Equal("fr.tag", pin.Name);
    //     Assert.Equal("some-tag", pin.Value);
    // }
}
```
