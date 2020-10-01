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

Thus, from the point of view of *data* there is no change to be made to the existing Cadmus Docker images.

Anyway, for the backend the usual way of building projects based on Cadmus is creating new parts specialized for that project, and providing an API customized to use them. The former task is specific to each project. The latter task can be completed by just composing Cadmus components together.

## Software

The API project includes only infrastructure and a RESTful API. All the core business and data logic are outside of it, in the Cadmus core project. API related services are in the API services library.

The pluggable components proper, i.e. parts and fragments, can be defined in any number of (typically) NuGet packages. So, any third party can provide its own components which can be plugged into the Cadmus architecture.

These pluggable components are bound to the API layer at design time, rather than later at runtime. You can build the API for your own components by either cloning the Cadmus API repository and modifying it, or creating a new ASP.NET web app from scratch and importing the required libraries. In both cases, you end up with an API which must serve a predefined set of plugged components.

### New Models

Adding a new set of models, i.e. parts and/or fragments, implies creating a new solution, by convention usually having a name starting with `Cadmus` (e.g. `CadmusItinera` for the *Itinera* project). The solution should include 3 new projects (with the following suggested naming conventions), to be distributed as NuGet packages:

1. `Cadmus.<NAME>.Parts`: class library with the model classes representing your parts and fragments. This is usually no more than a set of POCO classes with no logic, except for data pins generation where applicable.

2. `Cadmus.Seed.<NAME>.Parts`: class library with the seeders for the classes and fragments defined in the project at the previous step. This has as dependencies the core Cadmus seeder (`Cadmus.Seed`), and the corresponding parts/fragments project, and providing a seeder for each part and fragment defined there. Having a seeder next to each component is a recommended practice; it provides the system with the capability of creating mock data on the fly, and also helps designers better understand their models by facing the challenge of providing realistic data for them.

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

### New API

To build a new API, just create a new API project and reference all the required libraries, as specified in the following procedure:

1. create a new ASP.NET Core web API project.

2. add these NuGet packages:

- `AspNetCore.Identity.Mongo`
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
- `Swashbuckle.AspNetCore.Swagger`
- `Swashbuckle.AspNetCore.SwaggerGen`
- `Swashbuckle.AspNetCore.SwaggerUi`
- `Cadmus.Itinera.Models`
- `Cadmus.Itinera.Services`
- `Cadmus.Itinera.Controllers`

3. copy `Program.cs` from CadmusApi, adjusting the namespace and text messages as desired.

4. copy `Startup.cs` from CadmusApi, adjusting the namespace.

5. copy settings from `appsettings.json` in CadmusApi, adjusting application title and other app-dependent text as desired. Also, be sure to change:

- the database names (`DatabaseNames`;
- the Serilog connection string (`Serilog:ConnectionString`).

6. copy `Dockerfile` and `docker-compose.yml` from CadmusApi and adjust them for this project:

- in `Dockerfile`, change the project's name.
- in `docker-compose.yml`, change the `cadmus-api` image name, and its port (to reflect this project's port).

7. in the project's properties, ensure that the port is not conflicting with other port numbers in your environment (you can also see <https://stackoverflow.com/questions/37365277/how-to-specify-the-port-an-asp-net-core-application-is-hosted-on>).
