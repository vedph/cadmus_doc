# Plugins - Backend

## Data

Each project using Cadmus has its own databases, i.e.:

a) on *MongoDB*:

- data
- authentication and authorization
- auditing and diagnostic log

b) on *MySql*:

- data index. This is an automatic product, and can be fully rebuilt from the MongoDB data database. The name of this database is equal to the name of the MongoDB data database.

For all these databases, you can define the connection strings in the API settings. As these settings can be overridden by other sources like environment variables, it's easy to change them without having to change the existing backend image.

Thus, from the point of view of data there is no change to be made to the existing Cadmus Docker images.

## Software

The API project includes only infrastructure and a RESTful API. All the core business and data logic are outside of it, in the Cadmus core project. The pluggable components proper, i.e. parts and fragments, can be defined in any number of (typically) NuGet packages. So, any third party can provide its own components which can be plugged into the Cadmus architecture.

Yet, these components are bound to the API layer at design time, rather than later at runtime. To this end, what you have to do is cloning the API repository, adding or removing references to the components you want to use, and [rebuild the Docker image](../deploy/docker-build.md). The details of this process are outlined in the next section.

### Adding New Models

Adding a new set of models, i.e. parts and/or fragments, implies creating 3 new projects (with these suggested naming conventions), typically all in the same solution, to be distributed as NuGet packages:

1. `Cadmus.<NAME>.Parts`: class library with the model classes representing your parts and fragments. This is usually no more than a set of POCO classes with no logic, except for the data pins generation where applicable.

2. `Cadmus.Seed.<NAME>.Parts`: class library with the seeders for the classes and fragments defined in the project at the above step (1). This has as dependencies the core Cadmus seeder (`Cadmus.Seed`), and the corresponding parts/fragments project, and providing a seeder for each part and fragment defined there. Having a seeder next to each component is a recommended practice, as it provides the system with the capability of creating mock data on the fly, and also helps designers better understand their models by facing the challenge of providing realistic data for them.

3. `Cadmus.<NAME>.Services`: class library with the services to be used in the API infrastructure. This must add two simple services:

- an `IRepositoryProvider` implementation for the repository provider. Usually, all what you have to do is just copying the default `StandardRepositoryProvider`, and in its constructor ensure to add references to all the assemblies with the parts and fragments you are going to use (eventually removing those you are not going to use):

```cs
public StandardRepositoryProvider(IConfiguration configuration)
{
    // ...

    _map.Add(new[]
    {
        // Cadmus.Parts
        typeof(NotePart).GetTypeInfo().Assembly,
        // Cadmus.Lexicon.Parts
        typeof(WordFormPart).GetTypeInfo().Assembly,
        // Cadmus.Philology.Parts
        typeof(ApparatusLayerFragment).GetTypeInfo().Assembly
    });

    // ...
}
```

- an `IPartSeederFactoryProvider` implementation for the part seeder factory to be used with your components. Usually, all what you have to do is just copying the default `StandardPartSeederFactoryProvider`, and in its `GetFactory` method ensure to add references to all the assemblies with the parts and fragments you are going to use (eventually removing those you are not going to use):

```cs
public PartSeederFactory GetFactory(string profile)
{
    // ...

    // build the tags to types map for parts/fragments
    Assembly[] seedAssemblies = new[]
    {
        // Cadmus.Seed.Parts
        typeof(NotePartSeeder).Assembly,
        // Cadmus.Seed.Philology.Parts
        typeof(ApparatusLayerFragmentSeeder).Assembly
        // add other assemblies here...
    };

    // ...
}
```

### Modifying the API Project

In the API project:

1. add references to the NuGet packages `Cadmus.<NAME>.Parts`, `Cadmus.Seed.<NAME>.Parts`, and `Cadmus.<NAME>.Services`, as illustrated above.

2. in the `ConfigureServices` method of the `Startup` class replace the standard services classes with those using your services as provided in `Cadmus.<NAME>.Services` in these lines:

```cs
// repository
services.AddSingleton<IRepositoryProvider, StandardRepositoryProvider>();
// part seeder factory provider
services.AddSingleton<IPartSeederFactoryProvider,
    StandardPartSeederFactoryProvider>();
```

Once you have done this, just rebuild the project and the corresponding Docker image.
