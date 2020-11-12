# Development Environment

When developing on a single machine, I setup my environment as follows:

- a Docker container for MongoDB, named `mongo`.
- a Docker container for MySql, named `mysql`.
- [Visual Studio Community Edition](https://visualstudio.microsoft.com/vs/community/) for the backend.
- [NodeJS](https://nodejs.org/en/download/) and Angular for the frontend.
- [VSCode](https://code.visualstudio.com/download) for the frontend.

## Angular

To update to the latest version:

```ps1
npm uninstall -g @angular/cli
npm cache verify
npm install -g @angular/cli@latest
```

## MongoDB

**Windows**: to run a mongo container persisting data in the host via a volume in `c:\users\<USERNAME>\data\mongo`:

```ps1
docker run --name mongo -d -p 27017:27017 --volume c:/users/dfusi/dockerVolMongo/db:/data/db mongo --noauth
```

You can install Studio3T or MongoDB Compass to connect to the MongoDB service.

## MySql

**Windows**: to run a MySql container persisting data in the host via a volume in `c:\data\mssql`:

```ps1
docker run --name mssql -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=P4ss-W0rd!" -p 1433:1433 -v c:/data/mssql:/var/opt/mssql/data -d mcr.microsoft.com/mssql/server:2017-latest-ubuntu
```

You can connect to 127.0.0.1 with user=`sa` and password=`P4ss-W0rd!`.

The CLI tool for MSSQL, `sqlcmd`, is typically under `/opt/mssql-tools/bin`: run it with `-S localhost -U sa`.

You can install MySql Workbench only in your host and connect to the Docker service as specified.

## Switching Databases

By default, the Cadmus database used by the fronted is named `cadmus`. The backend API automatically creates this database if missing, together with all the others. Anyway, during development you typically have to deal with different databases while of course using the same codebase.

If you want to play in the frontend with different databases in the development environment, the typical procedure is:

1. *import the Cadmus MongoDB database* (via a custom tool, a standard import from a dump, etc.).

2. *index the imported database* at once using [cadmus-tool](https://github.com/vedph/cadmus_tool). The database name is just the MongoDB database name (e.g. `cadmus`); the profile file path is the full path to the JSON Cadmus profile file used for the imported database. Indexing normally happens during editing, but here we're importing a whole database at once, so we can use this tool to create the full corresponding index.

3. in the *web frontend*, temporarily edit `env.js` by changing the database name as desired. You can then launch both the backend and the frontend without changing anything else. The database used in the frontend (together with other relevant parameters) is defined in this `env.js`, which can be modified after the build. By default, it is `cadmus` too. The `env.js` file can be used to compile once, and then build different Docker images or scripts, each using a different version of that file.
