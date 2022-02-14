# Backend CLI Support

- [Backend CLI Support](#backend-cli-support)
  - [CLI Cadmus Repository Provider](#cli-cadmus-repository-provider)
  - [CLI Part Seeder Factory Provider](#cli-part-seeder-factory-provider)

To add support for the command-line interface of [Cadmus tool](https://github.com/vedph/cadmus_tool), whenever you create new parts in the backend you should also provide a library to be consumed as a plugin for that tool.

The CLI tool requires two factory providers for each new set of parts/fragments. These are essentially slight variations of the providers already implemented in the API services for each project (cf. `IRepositoryProvider` and `IPartSeederFactoryProvider`); yet, we currently keep this redundancy because for compatibility reasons Cadmus libraries have not yet upgraded to NET 6.0, which instead is the framework used in the CLI tool.

So you need these two CLI providers:

- **repository provider**, to create an instance of `ICadmusRepository`. This implements `ICliCadmusRepositoryProvider`.
- **part seeder factory provider**, to create an instance of `PartSeederFactory`. This implements `ICliPartSeederFactoryProvider`.

Templates follow for each of them.

## Creating Project

1. in your backend models solution, add a new NET 6 library project named `Cadmus.Cli.Plugin.__PRJ__` where `__PRJ__` is your project name (e.g. `Pura`).

2. whithin that solution, add reference to the parts project and to the seeder project.

3. add NuGet packages `Cadmus.Cli.Core`, `Cadmus.Mongo`.

4. add the two classes for the providers.

## CLI Cadmus Repository Provider

Template (replace `__PRJ__` with your project name):

```cs
using Cadmus.Cli.Core;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Core.Storage;
using Cadmus.Mongo;
using Cadmus.Parts.General;
using Cadmus.Philology.Parts;
using Cadmus.__PRJ__.Parts;
using Cadmus.Tgr.Parts.Codicology;
using Fusi.Tools.Config;
using System;
using System.Reflection;

namespace Cadmus.Cli.Plugin.__PRJ__
{
    /// <summary>
    /// CLI Cadmus repository provider for __PRJ__.
    /// Tag: <c>cli-repository-provider.__PRJ__</c>.
    /// </summary>
    /// <seealso cref="ICliCadmusRepositoryProvider" />
    [Tag("cli-repository-provider.__PRJ__")]
    public sealed class __PRJ__CliCadmusRepositoryProvider :
        ICliCadmusRepositoryProvider
    {
        private readonly IPartTypeProvider _partTypeProvider;

        /// <summary>
        /// Gets or sets the connection string.
        /// </summary>
        public string? ConnectionString { get; set; }

        /// <summary>
        /// Initializes a new instance of the
        /// <see cref="__PRJ__CliCadmusRepositoryProvider"/> class.
        /// </summary>
        public __PRJ__CliCadmusRepositoryProvider()
        {
            TagAttributeToTypeMap _map = new();
            _map.Add(new[]
            {
                // TODO: define all your libraries here...
                // typically you copy from Cadmus.__PRJ__.Services
                // e.g.:
                // Cadmus.Parts
                typeof(NotePart).GetTypeInfo().Assembly,
                // Cadmus.Philology.Parts
                typeof(ApparatusLayerFragment).GetTypeInfo().Assembly,
                // Cadmus.__PRJ__.Parts
                typeof(AnyOfYourLibraryPartsHere).GetTypeInfo().Assembly,
            });

            _partTypeProvider = new StandardPartTypeProvider(_map);
        }

        /// <summary>
        /// Creates the repository.
        /// </summary>
        /// <param name="database">The database.</param>
        /// <returns>Repository.</returns>
        /// <exception cref="ArgumentNullException">database</exception>
        public ICadmusRepository CreateRepository(string database)
        {
            if (database == null)
                throw new ArgumentNullException(nameof(database));

            // create the repository (no need to use container here)
            MongoCadmusRepository repository =
                new(_partTypeProvider, new StandardItemSortKeyBuilder());

            repository.Configure(new MongoCadmusRepositoryOptions
            {
                ConnectionString = string.Format(ConnectionString!, database)
            });

            return repository;
        }
    }
}
```

## CLI Part Seeder Factory Provider

Template:

```cs
using Cadmus.Cli.Core;
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.Parts.General;
using Cadmus.Seed.Philology.Parts;
using Cadmus.Seed.__PRJ__.Parts;
using Cadmus.Seed.Tgr.Parts.Grammar;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Fusi.Tools.Config;
using Microsoft.Extensions.Configuration;
using SimpleInjector;
using System;
using System.Reflection;

namespace Cadmus.Cli.Plugin.__PRJ__
{
    /// <summary>
    /// CLI part seeder factory provider for __PRJ__.
    /// Tag: <c>cli-seeder-factory-provider.__PRJ__</c>.
    /// </summary>
    /// <seealso cref="ICliPartSeederFactoryProvider" />
    [Tag("cli-seeder-factory-provider.__PRJ__")]
    public sealed class __PRJ__CliPartSeederFactoryProvider
        : ICliPartSeederFactoryProvider
    {
        public PartSeederFactory GetFactory(string profile)
        {
            if (profile == null)
                throw new ArgumentNullException(nameof(profile));

            // build the tags to types map for parts/fragments
            Assembly[] seedAssemblies = new[]
            {
                // TODO: your seeders here...
                // // typically you copy from Cadmus.__PRJ__.Services
                // e.g.:
                // Cadmus.Seed.Parts
                typeof(NotePartSeeder).Assembly,
                // Cadmus.Seed.Philology.Parts
                typeof(ApparatusLayerFragmentSeeder).Assembly,
                // Cadmus.Seed.__PRJ__.Parts
                typeof(WordFormsPartSeeder).GetTypeInfo().Assembly,
            };
            TagAttributeToTypeMap map = new();
            map.Add(seedAssemblies);

            // build the container for seeders
            Container container = new Container();
            PartSeederFactory.ConfigureServices(
                container,
                new StandardPartTypeProvider(map),
                seedAssemblies);

            container.Verify();

            // load seed configuration
            IConfigurationBuilder builder = new ConfigurationBuilder()
                .AddInMemoryJson(profile);
            var configuration = builder.Build();

            return new PartSeederFactory(container, configuration);
        }
    }
}
```
