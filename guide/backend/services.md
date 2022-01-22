# Backend Part Services

- [Backend Part Services](#backend-part-services)
  - [Adding Project](#adding-project)
  - [Adding Repository Provider](#adding-repository-provider)
  - [Adding Part Seeder Factory Provider](#adding-part-seeder-factory-provider)

Typically, when dealing with your own project having its parts, you also add a services library to provide API with ready to use services for getting a repository and a set of seeders configured for the parts to use in that project. These include all the parts you specifically add to your projects, and all the parts imported from other libraries. All these are glued into these services.

So, you can create the backend services library for your project in the same solution containing the parts specific to that project. If instead you are just importing parts from other libraries, typically adding the services directly to the API project is enough.

## Adding Project

1. in the solution created to contain your project own parts (which usually is named `Cadmus.PRJ`, and includes the parts library named `Cadmus.PRJ.Parts`), add a netstandard2.0 `Class Library` project named `Cadmus.PRJ.Services` where `PRJ` is your project's name.

2. remove `Class1.cs` from the project.

3. whithin the solution, add to this project references to the projects including your parts and their seeders.

4. add these NuGet packages:

```bash
install-package Cadmus.Core
install-package Cadmus.Index.Sql
install-package Cadmus.Mongo
install-package Fusi.Microsoft.Extensions.Configuration.InMemoryJson
```

5. also, add all the packages required to get the parts you want and their seeders. For instance:

```bash
install-package Cadmus.Seed.General.Parts
install-package Cadmus.Seed.Philology.Parts
```

6. typically you will also want to add some more metadata to the library, so that you can package it with NuGet. Open the project file (`.csproj`) and paste elements like these into the first `PropertyGroup`, changing their content accordingly, and save:

```xml
<IncludeSymbols>true</IncludeSymbols>
<SymbolPackageFormat>snupkg</SymbolPackageFormat>
<Authors>Daniele Fusi</Authors>
<Company>Fusi</Company>
<Product>Cadmus</Product>
<Description>Cadmus PURA API services.</Description>
<Copyright>by Daniele Fusi</Copyright>
<PackageLicenseExpression>GPL-3.0-or-later</PackageLicenseExpression>
<PackageTags>Cadmus;PURA</PackageTags>
<NeutralLanguage>en-US</NeutralLanguage>
<Version>1.0.0</Version>
```

## Adding Repository Provider

1. add a `PRJRepositoryProvider.cs` class with a content like this:

```cs
using System;
using System.Reflection;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Core.Storage;
using Cadmus.Mongo;
using Cadmus.__PRJ__.Parts;
using Microsoft.Extensions.Configuration;
using IConfiguration = Microsoft.Extensions.Configuration.IConfiguration;

namespace Cadmus.__PRJ__.Services
{
    /// <summary>
    /// Cadmus __PRJ__ repository provider.
    /// </summary>
    /// <seealso cref="IRepositoryProvider" />
    public sealed class __PRJ__RepositoryProvider : IRepositoryProvider
    {
        private readonly IConfiguration _configuration;
        private readonly IPartTypeProvider _partTypeProvider;

        /// <summary>
        /// Initializes a new instance of the <see cref="__PRJ__RepositoryProvider"/>
        /// class.
        /// </summary>
        /// <param name="configuration">The configuration.</param>
        /// <exception cref="ArgumentNullException">configuration</exception>
        public __PRJ__RepositoryProvider(IConfiguration configuration)
        {
            _configuration = configuration ??
                throw new ArgumentNullException(nameof(configuration));

            TagAttributeToTypeMap map = new TagAttributeToTypeMap();
            map.Add(new[]
            {
            	// TODO: add all the required parts here, e.g.:
                // Cadmus.General.Parts
                typeof(NotePart).GetTypeInfo().Assembly,
                // Cadmus.Philology.Parts
                typeof(ApparatusLayerFragment).GetTypeInfo().Assembly,
                // Cadmus.__PRJ__.Parts
                typeof(TheNameOfAPartFromYourOwnPartsLibrary).GetTypeInfo().Assembly,
            });

            _partTypeProvider = new StandardPartTypeProvider(map);
        }

        /// <summary>
        /// Gets the part type provider.
        /// </summary>
        /// <returns>part type provider</returns>
        public IPartTypeProvider GetPartTypeProvider()
        {
            return _partTypeProvider;
        }

        /// <summary>
        /// Creates a Cadmus repository.
        /// </summary>
        /// <returns>repository</returns>
        public ICadmusRepository CreateRepository()
        {
            // create the repository (no need to use container here)
            MongoCadmusRepository repository =
                new MongoCadmusRepository(
                    _partTypeProvider,
                    new StandardItemSortKeyBuilder());

            repository.Configure(new MongoCadmusRepositoryOptions
            {
                ConnectionString = string.Format(
                    _configuration.GetConnectionString("Default"),
                    _configuration.GetValue<string>("DatabaseNames:Data"))
            });

            return repository;
        }
    }
}
```

## Adding Part Seeder Factory Provider

1. add a `PRJPartSeederFactoryProvider.cs` class with a content like this:

```cs
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.__PRJ__.Parts;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Microsoft.Extensions.Configuration;
using SimpleInjector;
using System;
using System.Reflection;

namespace Cadmus.__PRJ__.Services
{
    /// <summary>
    /// __PRJ__ part seeders provider.
    /// </summary>
    /// <seealso cref="IPartSeederFactoryProvider" />
    public sealed class __PRJ__PartSeederFactoryProvider :
        IPartSeederFactoryProvider
    {
        /// <summary>
        /// Gets the part/fragment seeders factory.
        /// </summary>
        /// <param name="profile">The profile.</param>
        /// <returns>Factory.</returns>
        /// <exception cref="ArgumentNullException">profile</exception>
        public PartSeederFactory GetFactory(string profile)
        {
            if (profile == null)
                throw new ArgumentNullException(nameof(profile));

            // build the tags to types map for parts/fragments
            Assembly[] seedAssemblies = new[]
            {
            	// TODO: add your part seeder libraries here, e.g.:
                // Cadmus.Seed.General.Parts
                typeof(NotePartSeeder).Assembly,
                // Cadmus.Seed.Philology.Parts
                typeof(ApparatusLayerFragmentSeeder).Assembly,
                // Cadmus.Seed.__PRJ__.Parts
                typeof(TheNameOfAPartFromYourOwnSeedersLibrary).GetTypeInfo().Assembly,
            };
            TagAttributeToTypeMap map = new TagAttributeToTypeMap();
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

You can now package the library (typically, you can add it to the batch file already in place for packaging the part and part seeder libraries) and then consume it in your API project.
