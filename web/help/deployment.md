# Deployment Scenarios

This is an early prototype of the project, so the deployment experience is still raw; yet, you can take advantage of Docker to make it very easy. Here I just add some general guidance.

## Demo

The easiest scenario is for playing with Cadmus and a mock database.

You just need to install Docker in your target system, download a small Docker compose script and launch it. You can then browse to <localhost:4200>, login and try the editor.

This scenario is covered in more detail in the [quick start](quick-start).

## Customized Demo

A variant of the above scenario is using the demo with a _custom profile_. Using a custom profile you can define your items facets and parts as desired, and then let the system build a corresponding mock database for it.

This way you will be able to play with a demo which uses the same data models you plan to use.

The easiest way to specify the custom profile without having to rebuild a Docker image is just placing it somewhere in a web server. The profile is just a JSON file; so all what you have to do is uploading it, and let your Docker environment point to this resource:

1. create your own JSON profile. You can use the default profile as a model (you can find it in the Cadmus API project under `wwwroot/seed-profile.json`).
2. upload it somewhere in a web server.
3. edit the Docker compose script under the `cadmus-api` entry, by setting the environment variable `SEED__PROFILESOURCE` to the URI of your profile.
4. launch the Docker compose script as for the standard demo.

### Importing Data From JSON Dumps

You can also import some custom data at once to play with. At startup, you have the option of importing data (items and parts) from JSON dumps, after the seeding process has completed.

You can use this option to fully create a database when it does not exists, by allowing the seeder to seed the Cadmus profile without any items, and then importing JSON dumps from one or more files or HTTP(S) resources.

The import sources are specified in the `imports` section of the profile, just after the `seed` section, as a sibling of it; for instance:

```json
"imports": [
  "https://www.mysite.org/dumps/cadmus01.json",
  "https://www.mysite.org/dumps/cadmus02.json"
]
```

The files will be synchronously processed in the order they are specified.

These dumps are JSON files whose root element is always an array. The objects in the array can be either items with their `parts` property (which can be empty), or just parts.

This option can be used also with JSON files in the local file system, but this is rather an alternative targeted to developers.

Of course, you can always import data into the databases using the standard import methods, provided that you have access to the database server. If the database server is inside the Docker stack (see the next section), you can open its port to the host environment by just mapping it. You can then access the host and login into the database service from there. If instead it is outside the Docker stack, you can access the database machine independently of it.

## Production

For a production deployment you have a wide range of possibilities, according to the desired architecture. You first need to understand the layers in Cadmus and their usage.

The Docker compose script has 3 layers:

- `cadmus-mongo`: MongoDB database service. This is used for 3 different databases: one for Cadmus data, one for authorization data, and another for logging data. This layer has no dependencies.
- `cadmus-index`: MySql database service. This is used for a single database representing the index leveraged by Cadmus for searches inside the editor. This layer has no dependencies.
- `cadmus-api`: Cadmus API surface. This layer depends on both `cadmus-mongo` and `cadmus-index`.

### Data Services

For a production deployment you first have to decide whether you want to use external database services, or just keep these services inside the Docker compose stack.

Using dedicated database services is usually more performant and robust, as it distributes the workload across different machines. Of course, it requires you to have an existing infrastructure.

To replace the Docker-based service with your own, just remove the corresponding layer from the script, and change the corresponding connection string in the compose script. For instance, should you want to use an external MySql server:

- remove (or comment out) the cadmus-index layer;
- change the value of the connection string named `CONNECTIONSTRINGS__INDEX` in `cadmus-api/environment`.

If instead you are going to use data servers inside the Docker stack, you should set a volume for each of them, so that data are persisted outside the Docker container, in the host machine.

To this end, just add a `volumes` entry in the desired layer (see the [Docker documentation](https://docs.docker.com/compose/compose-file/#short-syntax-3)), e.g.:

```yml
volumes:
    - /host/data/source/path:/data/db
```

For a Windows host, the syntax would be like this:

```yml
volumes:
    - c:/users/dfusi/dockerVolMongo/db:/data/db
```

### Other Services

The API layer also requires an SMTP server to send email messages for managing users accounts. The API allows you to use your own mailing service: a generic SMTP using the .NET infrastructure, or third party services like SendGrid, Mailjet, etc.

Currently the demo is set to use a null-mailing service, which does nothing. It will be reconfigured once our own server infrastructure is in place, eventually providing options for picking the service in the configuration.

All the parameters for the SMTP configuration and the messages being built are defined via environment variables set in the Docker script; so you just have to change these values to fit your own infrastructure.

### Security

Stock users with their roles are defined in the standard [ASP.NET Core configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1), found in `appSettings.json` file, which can be overridden from a number of sources (typically environment variables).

**WARNING**: for obvious security reasons, you have to:

- change the passwords (and user names) of the stock users in the Docker script, where they can be set using environment variables. You can either set the passwords in the script, or better pass host environment variables through the Docker script to the API layer. This is accomplished by just removing the value from the variable name in the script; in this case, the Docker compose script will take the value from the host.
- change the JWT security key, defined in `appSettings.json` under `Jwt/SecureKey`.

All the `appSettings.json` settings can be overridden, typically from environment variables, which in turn can be set in the Docker script. So you can change all the settings as desired without having to rebuild any code or image.
