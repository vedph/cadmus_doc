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

9. copy and customize classes from `Services` in CadmusApi, so that these include the assemblies with your required parts and fragments.

10. in the project properties, set `swagger` as the value in `Launch browser`.

11. add as solution items a readme, and copy from CadmusApi `Dockerfile`, `docker-compose.yml`, `UpdateLocalPackages.bat`.
