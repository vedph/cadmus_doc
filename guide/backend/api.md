# Backend API

- [Backend API](#backend-api)
  - [Creating the Project](#creating-the-project)
  - [Settings](#settings)
  - [Program](#program)
  - [Startup](#startup)
  - [Assets](#assets)
  - [Docker](#docker)
  - [Readme](#readme)

The reference project for Cadmus API is [Cadmus API](https://github.com/vedph/cadmus_api).

## Creating the Project

1. create a new ASP.NET Core 6.0 web API project (no authentication) named `Cadmus<PRJ>Api`: select `None` for `Authentication type`, uncheck `Configure for HTTPS` and `Enable Docker`, check `Use controllers` and `Enable OpenAPI support`.

2. remove the mock `WeatherForecast.cs` class and its corresponding `WeatherForecastController.cs` class from the `Controllers` folder.

3. add these NuGet packages:

- `AspNetCore.Identity.Mongo`
- `Cadmus.Api.Controllers`
- `Cadmus.Api.Models`
- `Cadmus.Api.Services`
- `Microsoft.AspNetCore.Authentication.JwtBearer`
- `Microsoft.AspNetCore.Mvc.NewtonsoftJson`
- `Microsoft.Extensions.Configuration`
- `Newtonsoft.Json`
- `Polly`
- `Serilog`
- `Serilog.AspNetCore`
- `Serilog.Exceptions`
- `Serilog.Extensions.Hosting`
- `Serilog.Sinks.Console`
- `Serilog.Sinks.File`
- `Serilog.Sinks.MongoDB`
- `Swashbuckle.AspNetCore`
- your Cadmus part and seeders packages, e.g.:
  - `Cadmus.Seed.<PRJ>.Parts`
  - `Cadmus.<PRJ>.Services`

From inside Visual Studio, you can open the Package Manager Console and enter commands like:

```bash
install-package AspNetCore.Identity.Mongo
install-package Cadmus.Api.Controllers
install-package Cadmus.Api.Models
install-package Cadmus.Api.Services
install-package Microsoft.AspNetCore.Authentication.JwtBearer
install-package Microsoft.AspNetCore.Mvc.NewtonsoftJson
install-package Microsoft.Extensions.Configuration
install-package Newtonsoft.Json
install-package Polly
install-package Serilog
install-package Serilog.AspNetCore
install-package Serilog.Exceptions
install-package Serilog.Extensions.Hosting
install-package Serilog.Sinks.Console
install-package Serilog.Sinks.File
install-package Serilog.Sinks.MongoDB
install-package Swashbuckle.AspNetCore
```

## Settings

Add these settings to `appsettings.json` (replace `__PRJ__` with your project's name). Feel free to customize them as required. Please notice that all the sensitive data like users and passwords are there only for illustration purposes, and they will be overwritten by environment variables set in the hosting server.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Index": "Server=localhost;Database={0};Uid=root;Pwd=mysql;"
  },
  "DatabaseNames": {
    "Auth": "cadmus-__PRJ__-auth",
    "Data": "cadmus-__PRJ__"
  },
  "Serilog": {
    "ConnectionString": "mongodb://localhost:27017/{0}-log",
    "MaxMbSize": 10,
    "TableName": "Logs",
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Information",
        "System": "Warning"
      }
    }
  },
  "AllowedOrigins": [
    "http://localhost:4200",
  ],
  "Seed": {
    "ProfileSource": "%wwwroot%/seed-profile.json",
    "ItemCount": 100,
    "IndexDelay": 0
  },
  "Jwt": {
    "Issuer": "https://cadmus.azurewebsites.net",
    "Audience": "https://www.fusisoft.it",
    "SecureKey": "g>ueVcdZ7}:>4W5W"
  },
  "StockUsers": [
    {
      "UserName": "zeus",
      "Password": "P4ss-W0rd!",
      "Email": "dfusi@hotmail.com",
      "Roles": [
        "admin",
        "editor",
        "operator",
        "visitor"
      ],
      "FirstName": "Daniele",
      "LastName": "Fusi"
    }
  ],
  "Messaging": {
    "AppName": "Cadmus __PRJ__",
    "ApiRootUrl": "https://cadmus.azurewebsites.net/api/",
    "AppRootUrl": "https://fusisoft.it/apps/cadmus/",
    "SupportEmail": "webmaster@fusisoft.net"
  },
  "Editing": {
    "BaseToLayerToleranceSeconds": 60
  },
  "Indexing": {
    "IsEnabled": true,
    "DatabaseType": "mysql",
    "IsGraphEnabled": false
  },
  "Mailer": {
    "IsEnabled": false,
    "SenderEmail": "webmaster@fusisoft.net",
    "SenderName": "Cadmus __PRJ__",
    "Host": "",
    "Port": 0,
    "UseSsl": true,
    "UserName": "place in environment",
    "Password": "place in environment"
  }
}
```

## Program

Use this template to replace the code in `Program.cs` (replace `__PRJ__` with your project's name):

```cs
using System;
using System.Linq;
using System.Collections;
using System.Collections.Generic;
using Microsoft.AspNetCore.Hosting;
using Serilog;
using System.Diagnostics;
using Microsoft.Extensions.Hosting;
using Serilog.Events;
using Cadmus.Api.Services.Seeding;
using System.Threading.Tasks;
using Microsoft.Extensions.Configuration;
using Cadmus.Api.Services;

namespace Cadmus__PRJ__Api
{
    /// <summary>
    /// Program.
    /// </summary>
    public static class Program
    {
        private static void DumpEnvironmentVars()
        {
            Console.WriteLine("ENVIRONMENT VARIABLES:");
            IDictionary dct = Environment.GetEnvironmentVariables();
            List<string> keys = new();
            var enumerator = dct.GetEnumerator();
            while (enumerator.MoveNext())
            {
                keys.Add(((DictionaryEntry)enumerator.Current).Key.ToString()!);
            }

            foreach (string key in keys.OrderBy(s => s))
                Console.WriteLine($"{key} = {dct[key]}");
        }

        /// <summary>
        /// Creates the host builder.
        /// </summary>
        /// <param name="args">The arguments.</param>
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    // webBuilder.UseSerilog(); later
                });

        /// <summary>
        /// Entry point.
        /// </summary>
        /// <param name="args">The arguments.</param>
        public static async Task<int> Main(string[] args)
        {
            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Debug()
                .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
                .Enrich.FromLogContext()
                .WriteTo.Console()
#if DEBUG
                .WriteTo.File("cadmus-log.txt", rollingInterval: RollingInterval.Day)
#endif
                .CreateLogger();

            try
            {
                Log.Information("Starting Cadmus __PRJ__ API host");
                DumpEnvironmentVars();

                // this is the place for seeding:
                // see https://stackoverflow.com/questions/45148389/how-to-seed-in-entity-framework-core-2
                // and https://docs.microsoft.com/en-us/aspnet/core/migration/1x-to-2x/?view=aspnetcore-2.1#move-database-initialization-code
                var host = await CreateHostBuilder(args)
                    // add in-memory config to override Serilog connection string
                    // as there is no way of configuring it outside appsettings
                    // https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0#in-memory-provider-and-binding-to-a-poco-class
                    .ConfigureAppConfiguration((context, config) =>
                    {
                        IConfiguration cfg = AppConfigReader.Read();
                        string csTemplate = cfg.GetValue<string>("Serilog:ConnectionString");
                        string dbName = cfg.GetValue<string>("DatabaseNames:Data");
                        string cs = string.Format(csTemplate, dbName);
                        Debug.WriteLine($"Serilog:ConnectionString override = {cs}");
                        Console.WriteLine($"Serilog:ConnectionString override = {cs}");

                        Dictionary<string, string> dct = new()
                        {
                            { "Serilog:ConnectionString", cs }
                        };
                        // (requires Microsoft.Extensions.Configuration package
                        // to get the MemoryConfigurationProvider)
                        config.AddInMemoryCollection(dct);
                    })
                    .UseSerilog()
                    .Build()
                    .SeedAsync(); // see Services/HostSeedExtension

                host.Run();

                return 0;
            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "Cadmus __PRJ__ API host terminated unexpectedly");
                Debug.WriteLine(ex.ToString());
                Console.WriteLine(ex.ToString());
                return 1;
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }
    }
}
```

## Startup

Use this template for `Startup.cs` (replace `__PRJ__` with your project's name; in ASP.NET 6 web project you will need to add this file):

```cs
using System.Text;
using System.Text.Json;
using MessagingApi;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Extensions.Options;
using Microsoft.IdentityModel.Logging;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using AspNetCore.Identity.Mongo;
using IConfiguration = Microsoft.Extensions.Configuration.IConfiguration;
using System.Reflection;
using Serilog;
using Serilog.Events;
using Serilog.Exceptions;
using Cadmus.Core;
using Cadmus.Seed;
using Cadmus.Core.Config;
using Cadmus.Index.Config;
using Cadmus.Api.Services.Auth;
using Cadmus.Api.Services.Messaging;
using Cadmus.Api.Services;
using Microsoft.AspNetCore.HttpOverrides;
using Cadmus.Index.Graph;
using Cadmus.Index.MySql;
using Cadmus.Index.Sql;
using Cadmus.__PRJ__.Services;

namespace Cadmus__PRJ__Api
{
    /// <summary>
    /// Startup.
    /// </summary>
    public sealed class Startup
    {
        /// <summary>
        /// Gets the configuration.
        /// </summary>
        public IConfiguration Configuration { get; }

        /// <summary>
        /// Gets the host environment.
        /// </summary>
        public IHostEnvironment HostEnvironment { get; }

        /// <summary>
        /// Initializes a new instance of the <see cref="Startup"/> class.
        /// </summary>
        /// <param name="configuration">The configuration.</param>
        /// <param name="environment">The environment.</param>
        public Startup(IConfiguration configuration, IHostEnvironment environment)
        {
            Configuration = configuration;
            HostEnvironment = environment;
        }

        private void ConfigureOptionsServices(IServiceCollection services)
        {
            // configuration sections
            // https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/
            services.Configure<MessagingOptions>(Configuration.GetSection("Messaging"));
            services.Configure<DotNetMailerOptions>(Configuration.GetSection("Mailer"));

            // explicitly register the settings object by delegating to the IOptions object
            services.AddSingleton(resolver =>
                resolver.GetRequiredService<IOptions<MessagingOptions>>().Value);
            services.AddSingleton(resolver =>
                resolver.GetRequiredService<IOptions<DotNetMailerOptions>>().Value);
        }

        private void ConfigureCorsServices(IServiceCollection services)
        {
            string[] origins = new[] { "http://localhost:4200" };

            IConfigurationSection section = Configuration.GetSection("AllowedOrigins");
            if (section.Exists())
            {
                origins = section.AsEnumerable()
                    .Where(p => !string.IsNullOrEmpty(p.Value))
                    .Select(p => p.Value).ToArray();
            }

            services.AddCors(o => o.AddPolicy("CorsPolicy", builder =>
            {
                builder.AllowAnyMethod()
                    .AllowAnyHeader()
                    // https://github.com/aspnet/SignalR/issues/2110 for AllowCredentials
                    .AllowCredentials()
                    .WithOrigins(origins);
            }));
        }

        private void ConfigureAuthServices(IServiceCollection services)
        {
            // identity
            string connStringTemplate = Configuration.GetConnectionString("Default");

            services.AddIdentityMongoDbProvider<ApplicationUser, ApplicationRole>(
                options => { },
                mongoOptions =>
                {
                    mongoOptions.ConnectionString =
                        string.Format(connStringTemplate,
                        Configuration.GetSection("DatabaseNames")["Auth"]);
                });

            // authentication service
            services
                .AddAuthentication(options =>
                {
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
                })
                .AddJwtBearer(options =>
                {
                // NOTE: remember to set the values in configuration:
                // Jwt:SecureKey, Jwt:Audience, Jwt:Issuer
                IConfigurationSection jwtSection = Configuration.GetSection("Jwt");
                    string key = jwtSection["SecureKey"];
                    if (string.IsNullOrEmpty(key))
                        throw new InvalidOperationException("Required JWT SecureKey not found");

                    options.SaveToken = true;
                    options.RequireHttpsMetadata = false;
                    options.TokenValidationParameters = new TokenValidationParameters()
                    {
                        ValidateIssuer = true,
                        ValidateAudience = true,
                        ValidAudience = jwtSection["Audience"],
                        ValidIssuer = jwtSection["Issuer"],
                        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(key))
                    };
                });
#if DEBUG
            // use to show more information when troubleshooting JWT issues
            IdentityModelEventSource.ShowPII = true;
#endif
        }

        private static void ConfigureSwaggerServices(IServiceCollection services)
        {
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo
                {
                    Version = "v1",
                    Title = "API",
                    Description = "Cadmus __PRJ__ Services"
                });
                c.DescribeAllParametersInCamelCase();

                // include XML comments
                // (remember to check the build XML comments in the prj props)
                string xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
                string xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
                if (File.Exists(xmlPath))
                    c.IncludeXmlComments(xmlPath);

                // JWT
                // https://stackoverflow.com/questions/58179180/jwt-authentication-and-swagger-with-net-core-3-0
                // (cf. https://ppolyzos.com/2017/10/30/add-jwt-bearer-authorization-to-swagger-and-asp-net-core/)
                c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
                {
                    In = ParameterLocation.Header,
                    Description = "Please insert JWT with Bearer into field",
                    Name = "Authorization",
                    Type = SecuritySchemeType.ApiKey
                });
                c.AddSecurityRequirement(new OpenApiSecurityRequirement {
            {
                new OpenApiSecurityScheme
                {
                    Reference = new OpenApiReference
                    {
                        Type = ReferenceType.SecurityScheme,
                        Id = "Bearer"
                    }
                },
                Array.Empty<string>()
            }
            });
            });
        }

        /// <summary>
        /// Configures the services.
        /// </summary>
        /// <param name="services">The services.</param>
        public void ConfigureServices(IServiceCollection services)
        {
            // configuration
            ConfigureOptionsServices(services);

            // CORS (before MVC)
            ConfigureCorsServices(services);

            // base services
            services.AddControllers();
            // camel-case JSON in response
            services.AddMvc()
                // https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-2.2&tabs=visual-studio#jsonnet-support
                // Newtonsoft is required by MongoDB
                .AddNewtonsoftJson()
                .AddJsonOptions(options =>
                {
                    options.JsonSerializerOptions.PropertyNamingPolicy =
                        JsonNamingPolicy.CamelCase;
                });

            // authentication
            ConfigureAuthServices(services);

            // Add framework services
            // for IMemoryCache: https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory
            services.AddMemoryCache();

            // user repository service
            services.AddTransient<IUserRepository<ApplicationUser>,
                ApplicationUserRepository>();

            // messaging
            // TODO: you can use another mailer service here. In this case,
            // also change the types in ConfigureOptionsServices.
            services.AddTransient<IMailerService, DotNetMailerService>();
            services.AddTransient<IMessageBuilderService,
                FileMessageBuilderService>();

            // configuration
            services.AddSingleton(_ => Configuration);
            // repository
            services.AddSingleton<IRepositoryProvider, __PRJ__RepositoryProvider>();
            // part seeder factory provider
            services.AddSingleton<IPartSeederFactoryProvider,
                __PRJ__PartSeederFactoryProvider>();
            // item browser factory provider
            services.AddSingleton<IItemBrowserFactoryProvider>(_ =>
                new StandardItemBrowserFactoryProvider(
                    Configuration.GetConnectionString("Default")));
            // item index factory provider
            string indexCS = string.Format(
                Configuration.GetConnectionString("Index"),
                Configuration.GetValue<string>("DatabaseNames:Data"));
            services.AddSingleton<IItemIndexFactoryProvider>(_ =>
                new StandardItemIndexFactoryProvider(
                    indexCS));

            // graph repository
            services.AddSingleton<IGraphRepository>(_ =>
            {
                var repository = new MySqlGraphRepository();
                repository.Configure(new SqlOptions
                {
                    ConnectionString = indexCS
                });
                return repository;
            });

            // swagger
            ConfigureSwaggerServices(services);

            // serilog
            // Install-Package Serilog.Exceptions Serilog.Sinks.MongoDB
            // https://github.com/RehanSaeed/Serilog.Exceptions
            string maxSize = Configuration["Serilog:MaxMbSize"];
            services.AddSingleton<Serilog.ILogger>(_ => new LoggerConfiguration()
                .MinimumLevel.Information()
                .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
                .Enrich.WithExceptionDetails()
                .WriteTo.Console()
                /*.WriteTo.MSSqlServer(Configuration["Serilog:ConnectionString"],*/
                .WriteTo.MongoDBCapped(Configuration["Serilog:ConnectionString"],
                    cappedMaxSizeMb: !string.IsNullOrEmpty(maxSize) &&
                        int.TryParse(maxSize, out int n) && n > 0 ? n : 10)
                    .CreateLogger());
        }

        /// <summary>
        /// Configures the specified application.
        /// </summary>
        /// <param name="app">The application.</param>
        /// <param name="env">The environment.</param>
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            // https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-2.2#configure-a-reverse-proxy-server
            app.UseForwardedHeaders(new ForwardedHeadersOptions
            {
                ForwardedHeaders = ForwardedHeaders.XForwardedFor
                    | ForwardedHeaders.XForwardedProto
            });

            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                // https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0&tabs=visual-studio
                app.UseExceptionHandler("/Error");
                if (Configuration.GetValue<bool>("Server:UseHSTS"))
                {
                    Console.WriteLine("Using HSTS");
                    app.UseHsts();
                }
            }

            app.UseHttpsRedirection();
            app.UseRouting();
            // CORS
            app.UseCors("CorsPolicy");
            app.UseAuthentication();
            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });

            // Swagger
            app.UseSwagger();
            app.UseSwaggerUI(options =>
            {
                //options.SwaggerEndpoint("/swagger/v1/swagger.json", "V1 Docs");
                string url = Configuration.GetValue<string>("Swagger:Endpoint");
                if (string.IsNullOrEmpty(url)) url = "v1/swagger.json";
                options.SwaggerEndpoint(url, "V1 Docs");
            });
        }
    }
}
```

## Assets

Copy the whole `wwwroot` from [CadmusApi](https://github.com/vedph/cadmus_api), and customize its contents (the Cadmus profile, and if needed the messages template text).

Inside that folder, edit the `seed-profile.json` file, which contains the full Cadmus configuration for data and editors used in the project:

- remove all the parts, fragments, and seeders you do not use and all the parts, fragment, and seeders you require;
- do the same for thesaurus entries.

This is the core customization for the whole project. Usually, the profile file is created after the documentation is completed, and before creating the code.

Inside the `messages` folder you can customize the message templates as you prefer, but usually this is not required.

## Docker

1. in the project's root (where the `.sln` file is located), add a `Dockerfile` to build the Docker image (replace `__PRJ__` with your project's name):

```yml
# Stage 1: base
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Stage 2: build
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj", "Cadmus__PRJ__Api/"]
# copy local packages to avoid using a NuGet custom feed, then restore
# COPY ./local-packages /src/local-packages
RUN dotnet restore "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -s https://api.nuget.org/v3/index.json --verbosity n
# copy the content of the API project
COPY . .
# build it
RUN dotnet build "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -c Release -o /app/build

# Stage 3: publish
FROM build AS publish
RUN dotnet publish "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -c Release -o /app/publish

# Stage 4: final
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Cadmus__PRJ__Api.dll"]
```

2. add a `docker-compose.yml` file to allow you using the API in a composer stack (replace `PRJ` with your project name; of course, you can change your image name as required to fit your organization).

**ATTENTION**: under cadmus-api ports replace `59590` with the port value used by your API project (you can find it under the project's properties, Debug).

```yml
version: '3.7'

services:
  # MongoDB
  cadmus-mongo:
    image: mongo
    container_name: cadmus-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null # --quiet
    ports:
      - 27017:27017
    networks:
      - cadmus-network

  # MySql
  # https://github.com/docker-library/docs/tree/master/mysql#mysql_database
  # https://docs.docker.com/samples/library/mysql/#environment-variables
  # https://github.com/docker-library/mysql/issues/275 (troubleshooting connection)
  cadmus-index:
    image: mysql
    container_name: cadmus-index
    # https://github.com/docker-library/mysql/issues/454
    command: --default-authentication-plugin=mysql_native_password
    environment:
      # the password that will be set for the MySQL root superuser account
      # Note: use dictionary like here rather than array (- name = value)
      # or you might get MySql connection errors!
      # https://stackoverflow.com/questions/37459031/connecting-to-a-docker-compose-mysql-container-denies-access-but-docker-running/37460872#37460872
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_ROOT_HOST: '%'
    ports:
      - 3306:3306
    networks:
      - cadmus-network

  cadmus-api:
    image: vedph2020/cadmus_PRJ_api:1.0.0
    ports:
      # https://stackoverflow.com/questions/48669548/why-does-aspnet-core-start-on-port-80-from-within-docker
      - 59590:80
    depends_on:
      - cadmus-mongo
      - cadmus-index
    # wait for mongo before starting: https://github.com/vishnubob/wait-for-it
    command: ["./wait-for-it.sh", "cadmus-mongo:27017", "--", "dotnet", "CadmusTgrApi.dll"]
    environment:
      # for Windows use : as separator, for non Windows use __
      # (see https://github.com/aspnet/Configuration/issues/469)
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-mongo:27017/{0}
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-index;port=3306;Database={0};Uid=root;Pwd=mysql
      - SEED__INDEXDELAY=25
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
      - SENDGRID__ISENABLED=true
      - SENDGRID__SENDEREMAIL=info@cadmus.com
      - SENDGRID__SENDERNAME=cadmus
      - SENDGRID__APIKEY=todo
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
    networks:
      - cadmus-network

networks:
  cadmus-network:
    driver: bridge
```

3. add a `.dockerignore` file with this content:

```txt
**/.classpath
**/.dockerignore
**/.env
**/.git
**/.gitignore
**/.project
**/.settings
**/.toolstarget
**/.vs
**/.vscode
**/*.*proj.user
**/*.dbmdl
**/*.jfm
**/azds.yaml
**/bin
**/charts
**/docker-compose*
**/Dockerfile*
**/node_modules
**/npm-debug.log
**/obj
**/secrets.dev.yaml
**/values.dev.yaml
LICENSE
README.md
```

To build a Docker image (replace `PRJ` with your project's name):

```ps1
docker build . -t vedph2020/cadmus_PRJ_api:1.0.0 -t vedph2020/cadmus_PRJ_api:latest
```

## Readme

Add a Readme like this:

```txt
# Cadmus PRJ API

Quick Docker image build:

    docker build . -t vedph2020/cadmus_PRJ_api:1.0.0 -t vedph2020/cadmus_PRJ_api:latest

(replace with the current version).

This is a Cadmus API layer customized for the PRJ project. Most of its code is derived from shared Cadmus libraries. See the [documentation](https://github.com/vedph/cadmus_doc/blob/master/guide/api.md) for more.
```
