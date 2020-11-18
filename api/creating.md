# Creating a Cadmus API

1. create a new ASP.NET Core web API project.

2. add these packages:

- `AspNetCore.Identity.Mongo`
- `Cadmus.Api.Controllers`
- `Cadmus.Api.Models`
- `Cadmus.Api.Services`
- `Microsoft.AspNetCore.Authentication.JwtBearer`
- `Microsoft.AspNetCore.Mvc.NewtonsoftJson`
- `Polly`
- `Serilog`
- `Serilog.AspNetCore`
- `Serilog.Exceptions`
- `Serilog.Extensions.Hosting`
- `Serilog.Sinks.Console`
- `Serilog.Sinks.File`
- `Serilog.Sinks.MongoDB`
- `Swashbuckle.AspNetCore`

3. copy `Program.cs` from CadmusApi, adjusting the namespace and text messages as desired.

4. copy `Startup.cs` from CadmusApi, adjusting the namespace.

5. copy settings from `appsettings.json` in CadmusApi, adjusting application title and other app-dependent text as desired. Also, be sure to change:

- the database names (`DatabaseNames`);
- the Serilog connection string (`Serilog:ConnectionString`);
- the application name in messaging/mailing options.

Usually you would not want to change sensitive data in this file, which is anyway going to be published in a code repository. Rather, these data will be overridden by environment variables.

6. copy `Dockerfile` and `docker-compose.yml` from CadmusApi and adjust them for this project:

- in `Dockerfile`, change the project's name.
- in `docker-compose.yml`, change the `cadmus-api` image name, and its port (to reflect this project's port).

7. in the project's properties, ensure that the port is not conflicting with other port numbers in your environment (you can also see <https://stackoverflow.com/questions/37365277/how-to-specify-the-port-an-asp-net-core-application-is-hosted-on>).

8. copy `wwwroot` from CadmusApi, and customize its contents (the Cadmus profile, and if needed the messages template text).

9. copy and customize classes from `Services` in CadmusApi, so that these include the assemblies with your required parts and fragments. There are 2 required services:

- an `IRepositoryProvider` implementation for the repository provider. Usually, all what you have to do is just using the template below to create the `AppRepositoryProvider`. In its constructor ensure to add references to all the assemblies with the parts and fragments you are going to use (eventually removing those you are not going to use):

```cs
using System;
using System.Reflection;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Core.Storage;
using Cadmus.Mongo;
using Cadmus.Parts.General;
using Cadmus.Philology.Parts.Layers;
using Microsoft.Extensions.Configuration;
using IConfiguration = Microsoft.Extensions.Configuration.IConfiguration;

namespace Cadmus__PRJNAME__Api.Services
{
    /// <summary>
    /// Application's repository provider.
    /// </summary>
    public sealed class AppRepositoryProvider : IRepositoryProvider
    {
        private readonly IConfiguration _configuration;
        private readonly TagAttributeToTypeMap _map;
        private readonly IPartTypeProvider _partTypeProvider;

        /// <summary>
        /// Initializes a new instance of the <see cref="AppRepositoryProvider"/> class.
        /// </summary>
        /// <param name="configuration">The configuration.</param>
        /// <exception cref="ArgumentNullException">configuration</exception>
        public AppRepositoryProvider(IConfiguration configuration)
        {
            _configuration = configuration ??
                throw new ArgumentNullException(nameof(configuration));

            _map = new TagAttributeToTypeMap();
            _map.Add(new[]
            {
                // Cadmus.Parts
                typeof(NotePart).GetTypeInfo().Assembly,
                // Cadmus.Philology.Parts
                typeof(ApparatusLayerFragment).GetTypeInfo().Assembly,
                // TODO: add your additional parts here...
            });

            _partTypeProvider = new StandardPartTypeProvider(_map);
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
        /// <param name="database">The database name.</param>
        /// <returns>repository</returns>
        /// <exception cref="ArgumentNullException">null database</exception>
        public ICadmusRepository CreateRepository(string database)
        {
            if (database == null)
                throw new ArgumentNullException(nameof(database));

            // create the repository (no need to use container here)
            MongoCadmusRepository repository =
                new MongoCadmusRepository(
                    _partTypeProvider,
                    new StandardItemSortKeyBuilder());

            repository.Configure(new MongoCadmusRepositoryOptions
            {
                ConnectionString = string.Format(
                    _configuration.GetConnectionString("Default"), database)
            });

            return repository;
        }
    }
}
```

- an `IPartSeederFactoryProvider` implementation for the part seeder factory to be used with your components. Usually, you can just use the template below to build the `AppPartSeederFactoryProvider`. In its `GetFactory` method ensure to add references to all the assemblies with the parts and fragments you are going to use (eventually removing those you are not going to use):

```cs
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.Parts.General;
using Cadmus.Seed.Philology.Parts.Layers;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Microsoft.Extensions.Configuration;
using SimpleInjector;
using System;
using System.Reflection;

namespace Cadmus__PRJNAME__Api.Services
{
    /// <summary>
    /// Application's part seeders factory provider.
    /// </summary>
    public sealed class AppPartSeederFactoryProvider : IPartSeederFactoryProvider
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
                // Cadmus.Seed.Parts
                typeof(NotePartSeeder).Assembly,
                // Cadmus.Seed.Philology.Parts
                typeof(ApparatusLayerFragmentSeeder).Assembly,
                // TODO: add your parts here
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

10. in the project properties, set `swagger` as the value in `Launch browser`.

11. add as solution items a readme, and copy from CadmusApi `Dockerfile`, `docker-compose.yml`, `UpdateLocalPackages.bat`.
