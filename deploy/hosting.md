# Hosting a Cadmus Solution

This guide mostly targets system administrators. It is suggested to also review the page about [Cadmus API settings](settings.md) for more details about the available customizations.

Also, [here](./hosting-sample.md) you can find a real-world scenario hosting sample, using a simple Linux VM to host the whole Cadmus stack.

- [Hosting a Cadmus Solution](#hosting-a-cadmus-solution)
  - [Infrastructure](#infrastructure)
    - [Scenario 1: External Data Services](#scenario-1-external-data-services)
    - [Scenario 2: Dockerized Data Services](#scenario-2-dockerized-data-services)
  - [Databases](#databases)
    - [How Many Databases?](#how-many-databases)
  - [Security](#security)
    - [Auditing and Privacy](#auditing-and-privacy)
  - [CORS](#cors)
  - [Messaging](#messaging)
  - [Publishing Cadmus Services](#publishing-cadmus-services)

## Infrastructure

According to your deployment environment, you should decide if you want to use existing data services from your infrastructure, or just take advantage of those defined in the Cadmus default Docker stack.

The former is the commonest scenario, and requires you to derive a new Docker compose script from the existing one, by just removing the unused layers from the stack.

The latter scenario instead does not require this kind of customization. Anyway, you should use the compose script variant which makes use of an external volume to store data; otherwise, data would be destroyed with the container (e.g. when updating or otherwise mantaining it).

### Scenario 1: External Data Services

In this scenario, you have your own data services for MongoDB and/or MySql, and thus you need to remove the corresponding layers from the Docker compose script.

To this end:

1. remove or comment out the containers for MongoDB and MySql. Usually their names are `cadmus-mongo` and `cadmus-mysql`.

2. in the API container, remove the names of those containers from the `depends_on` array (or remove this array altogether if it becomes empty).

3. in the environment variables array, edit the connection strings to let them point to your data services:

- `CONNECTIONSTRINGS__DEFAULT` and `SERILOG__CONNECTIONSTRING` for MongoDB;
- `CONNECTIONSTRINGS__INDEX` for MySql.

The original script just uses container names as data server names in the connection strings. You will replace these with your data service address.

As the containers in the compose script all refer to a Docker virtual network, you must use `extra_hosts` to address a service outside this network; this is the case of your infrastructure data services. Thus, you should end up adding an `extra_hosts` section to each API service using a database, having an arbitrary name for your database service, and its mapping to an IP address. Once you have this, just replace the server name in the connection string with the arbitrary name chosen for your database service. To sum up, your API structure should resemble this:

```yml
services:
  the-api:
    image: ...
    environment:
      - CONNECTIONSTRINGS__DEFAULT=Server=vedphdb;...
    networks:
      - lsj-network
    extra_hosts:
      - "vedphdb:130.140.150.200"
```

In this API service the connection string points to the `vedphdb` server, which is mapped to the IP 130.140.150.200 in its `extra_hosts` section.

Please notice that in all the connection strings the `{0}` part is just a placeholder for the database name, which will be replaced by the API at runtime. So, you are effectively editing connection string _templates_. Of course, if required you can also change the database names (see [settings](./settings.md)).

4. when using an external MySql service, you can set to 0 the value of `SEED__INDEXDELAY`. This is the wait delay for the Cadmus API to connect to MySql. In a fully dockerized app this is required as MySql takes some time to start up, but you can set it to 0 (=no delay) when you have an independent data service.

### Scenario 2: Dockerized Data Services

In this scenario, you just have to pick the Docker compose script variant using an external volume for your data. This file usually is found in the GitHub repository next to the "standard" Docker compose script, with a `_linux-vol` suffix.

The only difference from the standard script is that database containers use a volume to store data, so you will find a `volumes` entry in their definition, like in these samples:

```yml
services:
  cadmus-mongo:
    image: mongo
    container_name: cadmus-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null
    ports:
      - 27017:27017
    volumes:
      - /var/db/mongo:/data/db
    networks:
      - cadmus-network

  cadmus-mysql:
    image: mysql
    container_name: cadmus-mysql
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_ROOT_HOST: "%"
    ports:
      - 3306:3306
    volumes:
      - /var/db/mysql:/var/lib/mysql
    networks:
      - cadmus-network
```

You must ensure that on your host computer you have created the folders to be used by the containers. The host directory is the one before the colon in the volumes entry.

Also, if for any reason your host machine is already running MySql or MongoDB on their default ports, you must change their ports in the Docker compose script, so that they get remapped to some other value which does not conflict with your host machine.

With this solution, you can safely destroy your container for whatever reason, and still keep your data safe in the volume. Once you start a new container, it will pick up the same data.

Anyway, don't rely on the volume files only; it is suggested that you also regularly make dumps of your databases, by directly connecting to their containers (<localhost:27017> for MongoDB and <localhost:3306> for MySql, unless you have changed these ports in the script).

If instead you are just experimenting, you can safely use the "standard" Docker compose script with no volume. In this case, your data is kept only until you destroy the container (e.g. via a `docker-compose down`; just stopping and restarting the container will instead preserve your data). In this case, you will start anew with fresh databases.

## Databases

When the Cadmus API starts, it connects to its data services and checks whether its required databases exist. If they do not exist, they get created, and usually seeded with mock data.

If you want to start with empty databases, set `Seed:ItemCount` to 0. In this way, databases will be created empty, which is usually what is desired in a true editing environment.

### How Many Databases?

Cadmus uses a number of MongoDB and MySql databases. At a minimum, they are:

- 1 MongoDB database for data.
- 1 MongoDB database for user accounts.
- 1 MongoDB database for auditing.
- 1 MySql database for real-time data indexing.

Anyway, depending on the Docker stack, more databases might be used. For instance, in Cadmus _Itinera_ we add another MySql database to hold a project-wide, centrialized bibliography.

## Security

The source code and Docker scripts include some default, dummy values for sensitive data like connection strings, user accounts, etc. It is imperative that you change all these settings before deploying a Cadmus based solution.

For the details of the Cadmus API settings you can refer to [this page](settings.md); here I just list the sensitive settings.

All these settings get overridden by setting environment variables on the server side. In the context of a Docker compose script, this is usually done inside the script itself. Of course, a script customized in this way should never leave the boundaries of your host machine.

At a minimum, you should change:

- `ConnectionStrings:Default`
- `ConnectionStrings:Index`
- `Serilog:ConnectionString`
- `Jwt:SecureKey`
- `Mailer:UserName`
- `Mailer:Password`
- `StockUsers`: all the accounts. Cadmus provides a set of builtin user accounts, with various levels of privilege. You should at least provide an admin builtin account.

Please notice that when referring to these settings keys in the Docker compose script you will use `__` (2 underscore characters, not just 1!) instead of colons (and usually capitalize the keys). Thus, `Jwt:SecureKey` becomes `JWT__SECUREKEY`.

You will find that some of these keys are already present in the Docker compose script, to give you a guidance by sample. You just have to replace their values, and add other entries in the same way.

### Auditing and Privacy

Cadmus has a granular auditing policy for data being edited. Most edits are logged in the auditing log (hosted in a MongoDB database), and the full editing history of each datum is stored in the data themselves. Please take the appropriate measures to periodically check this log for suspected activities, and protect it to comply with privacy requirements.

Anyway, no personal data is directly found in the log, as it just stores user names, which usually mean nothing outside a team. For instance, my user name for testing is "zeus"; and the mapping between zeus and my real name, if any, is found only in another database, related to user accounts. Of course, it's up to you to decide whether you want to map user names to real names, or just use first names, fake names, or whatever you prefer.

In any case, all sensitive operations on data are logged with user names and their IP address. The log is cyclic, so that it won't grow indefinitely (usually it's limited to 10 MB); you can anyway control its options via `Serilog`-related settings in the configuration.

Data history instead never gets pruned, as it's part of the data themselves and can be used to recover from errors or other accidents, and track the evolution of data being entered.

## CORS

Cadmus API implement [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) for cross-origin access. You can configure the allowed origins for CORS under the `AllowedOrigins` section of the API settings.

Usually, all what you have to do is set the `ALLOWEDORIGINS__0` environment variable to the URL of your web app. Allowed origins is an array, so that the `_0` suffix just means that you are setting the first item in it. The order in the array does not matter.

As for API access, the web frontend essentially requires two changes (see the [sample](./hosting-sample.md#docker_compose_script)):

1. compile a production version of the web app and then change the `env.js` constants, so that the API URL points to the location you are using for the backend. The default location is `localhost`, which is meaningful only when launching Cadmus locally on your own machine. Usually I create a production image with just this change, which does not require recompiling the web app (you just edit `env.js` found in the compiled files); conventionally, its tag is equal to the non-production app, with a `-prod` suffix.

2. add to the CORS allowed origins the URL of your web app.

## Messaging

The Cadmus API can optionally use some messaging functions to communicate with registered users. Usually, the only motivation for this is enabling password recovery functions, users creation (by administrators), and similar user-driven maintenance tasks.

The default API settings anyway have messaging disabled, as you should otherwise provide an SMTP account to be used.

If you want to enable messaging, you must change settings in the `Mailer` section. By default, Cadmus API use a generic SMTP account to send such messages, as they are sparingly used and thus do not require mass-mailing services. If anyway you want to use such services, Cadmus provides two builtin providers for [SendGrid](www.sendgrid.com) and [Mailjet](www.mailjet.com), or you can add your own providers.

The messaging provider is usually chosen when creating the API for a specific project. It just implies replacing a single line in the code, to tell the dependency injection system which mailer component should be used for messaging.

## Publishing Cadmus Services

Once you have setup your Docker compose script for Cadmus, you just have to launch it in your hosting environment, and decide which Cadmus services you want to expose to the outer world.

To quickly launch the script you can just enter the folder where you placed your `docker-compose.yml` file and run:

```bash
docker-compose up
```

You can then destroy the containers with `docker-compose down`. Most times anyway you will rather host the Docker stack into some more advanced environment.

As for publishing, usually you just expose the surface layer, i.e. the web application with the editor. In some cases, you might also want to expose the API layer.

Unless you change the default ports in the Docker compose script, you will find these endpoints:

- Cadmus API service can be browsed at <localhost:X/swagger> where `X` is the port number specified in your Docker compose script for the API container.
- Cadmus application at <localhost:4200>.
