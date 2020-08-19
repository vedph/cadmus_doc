# Cadmus API Settings

The Cadmus API settings are found in `appsettings.json`, as for any typical .NET Core web API application.

## Cadmus-Specific Sections

All these sections are located at the root level of the configuration hierarchy, i.e. in JSON they are direct children of the root object:

- `ConnectionStrings`: object where each key is the name of a connection string:
  - `Default`: it rather is a connection string template, pointing to the MongoDB server. The database name is a placeholder represented by `{0}` in that string, and will be set at runtime.
  - `Index`: the connection string template for a RDBMS database for the Cadmus index. Currently this is a MySql database. The database name is a placeholder represented by `{0}` in that string, and will be set at runtime.
- `DatabaseNames`: the names of the two main Cadmus databases to be used by the API:
  - `Data`: the name of the Cadmus data database.
  - `Auth`: the name of the Cadmus authentication and authorization database.
- `Serilog`: the [Serilog](https://serilog.net/)-based logging configuration.
- `Seed`: data seed profile and desired items count. This is used for seeding a newly created database when the API starts up. In the profile path you can use variables between `%`. The `%wwwroot%` variable is reserved to represent the web content root directory; any other variable name (excluding the `%` delimiters) is resolved from the app's configuration (or just removed when no resolution is possible). For more information about seeding, see the documentation in the backend repository (`cadmus_core`).
  - `ProfileSource`: the source for the JSON profile file.
  - `ItemCount`: the count of desired items in the database.
  - `IndexDelay`: the delay (in seconds) to allow after seeding items has completed, before starting to index them.
- `Jwt`: JWT tokens configuration:
  - `Issuer`: the issuer.
  - `Audience`: the audience.
  - `SecureKey`: this value is a mock used for development. Replace this with your own.
- `StockUsers`: a set of stock users which get seeded into the authentication database when not found. Of course, the authentication data here is just for development purposes, and you should replace them as desired. This is an array, where each item is an object with these properties:
  - `UserName`: the username.
  - `Password`: the password. Of course, the passwords you find in the configuration file are mock passwords using during development. True password are outside of the source code, and get set via environment variables in the host server.
  - `Email`: the user email address.
  - `Roles`: an array with all the user's roles. Roles can be picked among `admin`, `editor`, `operator`, and `visitor`. A user can have any number of roles.
  - `FirstName`: user's first name.
  - `LastName`: user's last name.
- `Messaging`: messaging configuration. Messaging here refers to the component in charge of building the message's text, rather than sending it. Messaging is used for administrative or maintenance purposes. For instance, email messages are used to allow users retrieve a forgotten password. Properties are:
  - `AppName`: the application name which appears in the message's text.
  - `ApiRootUrl`: the root URL to the API services. This is used to build clickable links in messages.
  - `AppRootUrl`: the root URL to the web application frontend. This is used to build clickable links in messages.
  - `SupportEmail`: the email address which should appear as the message's sender.
- `Mailer`: mailing service configuration. The mailer service is used to send messages via email. The settings here depend on the service used. For the "standard" SMTP service they are:
  - `IsEnabled`: enable or disable mailing. When disabled, no email message will ever be sent.
  - `SenderEmail`: the email address to appear as the subject of sent emails.
  - `SenderName`: the name to appear as the sender's name of sent emails.
  - `Host`: the SMTP host address.
  - `Port`: the SMTP host port.
  - `UseSsl`: true to use SSL.
  - `UserName`: the username for the SMTP service.
  - `Password`: the password for the SMTP service. As for any other sensitive data, the value here is provided for development purposes only, and you should override it.
- `Editing`: editing configuration:
  - `BaseToLayerToleranceSeconds`: the tolerance interval, expressed in seconds, between the save time of a text part and that of its layer part. If not specified, a default value of 60 is used. This is a parameter used when detecting potential breaks in the layer parts when their base text gets edited. A layer part is potentially broken when the corresponding text part has been saved (with a different text) either after it, or a few time before it; this interval specifies that time. In both cases, this implies that the part fragments might have broken links, as the underlying text was in some way changed. To detect a potential break we can just check for last modified date and time; if the above conditions for save date and time are not met, the method can return false. If instead they are met, we must ensure that text has changed. To this end, we must go back in the text part history to find the latest save which changed the text, and refer to its date and time.
- `Indexing`: data indexing configuration:
  - `IsEnabled`: true to enable indexing. It is recommended to keep this true, otherwise the frontend will not have access to search capabilities inside the editor.
  - `DatabaseType`: currently set to `mysql`. At present we provide support for MySql (and MariaDB) and SQL Server (`mssql`), but the only option for the configuration here is `mysql` to avoid unnecessary dependencies.

## Service-Specific Sections

All these sections are located at the root level of the configuration hierarchy, i.e. in JSON they are direct children of the root object.

### Auditing

- `Serilog`: [Serilog](https://serilog.net/)-specific logging configuration. Currently we log on a MongoDB database, so that typically the connection string found inside this section will be equal to the default connection string template, except that it points to a specific, named database.

### Messaging

Messaging happens via email. The configuration depends on the mailing service you choose. Currently, the following SMTP services are supported, but only the first is provided in the API configuration to avoid importing unnecessary modules:

- a general SMTP server account.
- [SendGrid](www.sendgrid.com): the SMTP configuration should be rooted under a property named `SendGrid`.
- [MailJet](www.mailjet.com): the SMTP configuration should be rooted under a property named `MailJet`.

Currently we are using the general SMTP server account. You can import modules for other providers and replace the service and options.

All the messaging services share these options:

- `IsEnabled`: a boolean used to enable or disable messaging.
- `SenderEmail`: the sender email address.
- `SenderName`: the sender human-friendly name.

Other options are specific for each service. Of course, any sensitive configuration data found here should be fake. True passwords will be set as environment variables in the host.

(a) general SMTP server:

- `Host`: the SMTP host.
- `Port`: the SMTP host port number.
- `UseSsl`: boolean telling whether we should use SSL for SMTP.
- `UserName`: the SMTP account username.
- `Password`: the SMTP account password.

(b) SendGrid:

- `ApiKey`: the SendGrid API key.

(c) MailJet:

- `ApiKey`: the MailJet API key.
- `ApiSecret`: the MailJet API secret.

## Environment Variables

To override settings you typically use environment variables in the host. Every sensitive setting is systematically overridden; other settings can be overridden at will, to customize the API's behavior and fit it into its hosting environment.

The Cadmus API uses the standard ASP.NET Core 3 [default configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#default-configuration) order, namely (for app configuration):

1. settings from `appsettings.json`;
2. settings from the environment-specific versions of `appsettings.json` (named like `appsettings.{Environment}.json`);
3. secret manager (in the `Development` environment); this relies on a user secrets file, a JSON file stored on the local developer's machine, outside of the source directory (and thus of source control);
4. environment variables;
5. command-line arguments.

The last override wins. Thus, the command line arguments have the highest precedence, followed by environment variables, etc.

Each setting in the JSON hierarchy gets addressed through a flat "path", using as a separator `:` for Windows, or `__` for non-Windows hosts where the semicolon might not work in environment variables. The double underscore is supported by all platforms, and is automatically converted into a colon. Such paths are case insensitive.

For instance, the `Default` connection string under `ConnectionStrings` can be addressed via `CONNECTIONSTRINGS__DEFAULT`.

When addressing array's items, you can use the item's index as a part of the path. For instance, to address the `Password` property of the first user among stock users (`StockUsers` property) you can use `STOCKUSERS__0__PASSWORD`, where `0` is the index to the first array's item, much like `stockUsers[0].password` in JS.

As for [command line](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#command-line-configuration-provider), these are the main points:

- the argument is a key=value pair. The value (which can be empty) either follows the `=` sign, or a space when the key is prefixed with `--` or `/`.
- do not mix the two alternative syntaxes (with `=` or prefix).

Samples from the quoted documentation:

```bash
dotnet run CommandLineKey1=value1 --CommandLineKey2=value2 /CommandLineKey3=value3
dotnet run --CommandLineKey1 value1 /CommandLineKey2 value2
dotnet run CommandLineKey1= CommandLineKey2=value2
```
