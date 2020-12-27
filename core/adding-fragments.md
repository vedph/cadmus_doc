# Adding Fragments

For seeder templates see [seeding](seeding.md).

## Fragment Template

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Fusi.Tools.Config;
using Cadmus.Core.Layers;
using Cadmus.Core;

namespace Cadmus.__PROJECT__.Parts
{
    /// <summary>
    /// TODO fragment.
    /// Tag: <c>fr.it.vedph.__PROJECT__.__NAME__</c>.
    /// </summary>
    /// <seealso cref="ITextLayerFragment" />
    [Tag("fr.it.vedph.__PROJECT__.__NAME__")]
    public sealed class __NAME__LayerFragment : ITextLayerFragment
    {
        /// <summary>
        /// Gets or sets the location of this fragment.
        /// </summary>
        /// <remarks>
        /// The location can be expressed in different ways according to the
        /// text coordinates system being adopted. For instance, it might be a
        /// simple token-based coordinates system (e.g. 1.2=second token of
        /// first block), or a more complex system like an XPath expression.
        /// </remarks>
        public string Location { get; set; }

        // TODO: add properties

        /// <summary>
        /// Get all the key=value pairs exposed by the implementor.
        /// </summary>
        /// <param name="item">The optional item. The item with its parts
        /// can optionally be passed to this method for those parts requiring
        /// to access further data.</param>
        /// <returns>The pins: <c>fr.tag</c>=tag if any.</returns>
        public IEnumerable<DataPin> GetDataPins(IItem item = null)
        {
            // TODO: build pins, eventually using DataPinBuilder like this:
            //DataPinBuilder builder = new DataPinBuilder(
            //    new StandardDataPinTextFilter());

            //// fr-tot-count
            //builder.Set(PartBase.FR_PREFIX + "tot", Entries?.Count ?? 0, false);

            //return builder.Build(null);

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
        public IList<DataPinDefinition> GetDataPinDefinitions()
        {
            return new List<DataPinDefinition>(new[]
            {
                // TODO: replace this with your pins definitions
                new DataPinDefinition(DataPinValueType.String,
                    PartBase.FR_PREFIX + "some-pin-name",
                    "The pin description.")
            });
        }

        /// <summary>
        /// Returns a <see cref="string" /> that represents this instance.
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
}
```

## Test Template

```cs
using Cadmus.Core;
using Cadmus.Seed.Tgr.Parts.Grammar;
using Cadmus.Tgr.Parts.Grammar;
using Fusi.Tools.Config;
using System;
using System.Linq;
using System.Collections.Generic;
using System.Reflection;
using System.Text;
using Xunit;

namespace Cadmus.__PROJECT__.Parts
{
    public sealed class __NAME__LayerFragmentTest
    {
        private static __NAME__LayerFragment GetFragment()
        {
            var seeder = new __NAME__LayerFragmentSeeder();
            return (__NAME__LayerFragment)
                seeder.GetFragment(null, "1.2", "exemplum fictum");
        }

        private static __NAME__LayerFragment GetEmptyFragment()
        {
            return new __NAME__LayerFragment
            {
                Location = "1.23",
                // TODO: set fragments data here...
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
            __NAME__LayerFragment fragment = GetFragment();

            string json = TestHelper.SerializeFragment(fragment);
            __NAME__LayerFragment fragment2 =
                TestHelper.DeserializeFragment<__NAME__LayerFragment>(json);

            Assert.Equal(fragment.Location, fragment2.Location);
            // TODO: check fragments data here...
        }

        [Fact]
        public void GetDataPins_Tag_1()
        {
            __NAME__LayerFragment fragment = GetEmptyFragment();
            List<DataPin> pins = fragment.GetDataPins(null).ToList();

        	// TODO assert pins: e.g.
	        // fr-tot-count
            // DataPin pin = pins.Find(p => p.Name == PartBase.FR_PREFIX + "tot-count");
            // Assert.NotNull(pin);
            // Assert.Equal("3", pin.Value);

            // fr-tag
            // pin = pins.Find(p => p.Name == PartBase.FR_PREFIX + "tag"
            //    && p.Value == "odd");
            // Assert.NotNull(pin);
        }
    }
}
```

You can add this `TestHelper`:

```cs
using Cadmus.Core;
using Cadmus.Core.Layers;
using System;
using System.Text.Json;
using Xunit;

namespace Cadmus.__PROJECT__.Parts.Test
{
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
    }
}
```
