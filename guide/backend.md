# Backend Models

- [Backend Models](#backend-models)
  - [Requirements](#requirements)
  - [Creating Solution](#creating-solution)
  - [Adding Parts or Fragments](#adding-parts-or-fragments)
  - [Adding Part or Fragment Seeders](#adding-part-or-fragment-seeders)

The backend is a set of C# libraries, built with VS. This step is required only if you have new data models (parts or fragments) specific to your project.

The following procedure will:

- create a new Visual Studio solution for the backend components (business layer).
- add to it a library for parts/fragments (if required), another library for their mock data seeders, and a third library for API services. Also, each model-related library will have its unit tests library.

## Requirements

- Visual Studio Community Edition or higher.

## Creating Solution

1. launch VS and create a new _blank solution_ named `Cadmus<PRJ>`.

2. add to this solution a _C# .NET Standard class library_, named `Cadmus.<PRJ>.Parts`. This will hold parts and fragments specific to your projects. Usually a single library is enough, but you are free to distribute components across several libraries should you need more granularity for their reuse. Once created, delete the empty `Class1.cs` file from it.

![adding new project](./img/a01_add-new-project.png)

![adding new project](./img/a02_add-new-project.png)

3. add another _C# .NET Standard class library_ named `Cadmus.Seed.<PRJ>.Parts` to provide the mock data seeders for your components. This is not strictly a requirement, but it's suggested to let you play with the editor while building it. Once created, delete the empty `Class1.cs` file from it.

4. add another _C# .NET Standard class library_ named `Cadmus.<PRJ>.Services` to provide some API services to plug into your API. Once created, delete the empty `Class1.cs` file from it.

5. add a _XUnit Test Project_ named `Cadmus.<PRJ>.Parts.Test` to contain the tests for the `Cadmus.<PRJ>.Parts` library. Alternatively, any other unit test framework can be used; this just reflects my preferences, and is suggested as the test templates I provide use XUnit. Once created, delete the empty `UnitTest1.cs` class.

![adding new project](./img/a03_add-new-xunit-project.png)

6. add a _XUnit Test Project_ named `Cadmus.Seed.<PRJ>.Parts.Test` to contain the tests for the `Cadmus.Seed.<PRJ>.Parts` library. Alternatively, any other unit test framework can be used; this just reflects my preferences, and is suggested as the test templates I provide use XUnit. Once created, delete the empty `UnitTest1.cs` class.

Your solution should now look like this (here `<PRJ>` is `Pura`):

![adding new project](./img/a04_solution.png)

7. for both the test projects created above (5-6), add a reference to their test target library. This can be done by right clicking the `Dependencies` node under the test project, selecting `Add Project Reference` from the popup menu, and checking the target project in the list which appears. Finally close the dialog with `OK`.

![adding new project](./img/a05_project-deps.png)

![adding new project](./img/a06_project-deps.png)

8. as your seeders need to know about the parts/fragments they are creating, add a project reference targeting the `Cadmus.PRJ.Parts` project to the `Cadmus.Seed.PRJ.Parts` project.

## Adding Parts or Fragments

You can now add as many parts and fragments as required to the `Cadmus.<PRJ>.Parts` project.

1. add a reference to the Cadmus core components to this project. This can be done in the VS UI, by adding a new NuGet package named `Cadmus.Core`, or by editing the `csproj` project XML file, and adding this line under an `<ItemGroup>` element (replace the version number with the latest available version):

```xml
<PackageReference Include="Cadmus.Core" Version="2.3.5" />
```

Should you need existing components to build your own (e.g. to extend or integrate them), add their packages in the imports too.

2. add a plain C# class for each part or fragment, representing its data model. Please refer to these pages for details:

- [adding parts](../core/adding-parts.md)
- [adding fragments](../core/adding-fragments.md)

## Adding Part or Fragment Seeders

For each part or fragment you should provide a corresponding mock data seeder to the `Cadmus.Seed.<PRJ>.Parts` project. This is extremely useful to let developers and users play with the editor.

1. add a reference to the Cadmus core seed components to this project, as explained above. Also, a reference to the `Bogus` package is useful to leverage the power of this library rather than creating mock data from scratch. The package references are listed below (replace the version number with the latest available version):

```xml
<PackageReference Include="Bogus" Version="32.1.1" />
<PackageReference Include="Cadmus.Core" Version="2.3.5" />
<PackageReference Include="Cadmus.Seed" Version="1.1.7" />
```

2. add a plain C# class for each part or fragment seeder. Please refer to these pages for details:

- [adding parts](../core/adding-parts.md)
- [adding fragments](../core/adding-fragments.md)

TODO: complete
