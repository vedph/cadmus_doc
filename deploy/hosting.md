# Hosting a Cadmus Solution

- [Hosting a Cadmus Solution](#hosting-a-cadmus-solution)
  - [Infrastructure](#infrastructure)
    - [Scenario 1: External Data Services](#scenario-1-external-data-services)
    - [Scenario 2: Dockerized Data Services](#scenario-2-dockerized-data-services)
  - [Security](#security)

## Infrastructure

Accordig to your deployment environment, you should decide if you want to use existing data services from your infrastructure, or just take advantage of those defined in the Cadmus default Docker stack.

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

The original script just uses container names as data server names in the connection strings. You will replace these with your data service address (usually, an IP address).

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
      MYSQL_ROOT_HOST: '%'
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

## Security

The source code and Docker scripts include some default, dummy values for sensitive data like connection strings, user accounts, etc. It is imperative that you change all these settings before deploying a Cadmus based solution.

For the details of the Cadmus API settings you can refer to [this page](settings.md); here I just list the sensitive settings.

All these settings get overridden by setting environment variables on the server side. In the context of a Docker compose script, this is usually done inside the script itself. Of course, a script customized in this way should never leave the boundaries of your host machine.

At a minimum, you should change:

- `ConnectionStrings:Default`
- `ConnectionStrings:Index`
- `Serilog:ConnectionString`
- `Jwt:SecureKey`
- `StockUsers`: all the accounts
- `Mailer:UserName`
- `Mailer:Password`

Please notice that when referring to these settings keys in the Docker compose script you will use `__` (2 underscore characters, not just 1!) instead of colons (and usually capitalize the keys). Thus, `Jwt:SecureKey` becomes `JWT__SECUREKEY`.

You will find that some of these keys are already present in the Docker compose script, to give you a guidance by sample. You just have to replace their values, and add other entries in the same way.
