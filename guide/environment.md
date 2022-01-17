# Development Environment

When developing on a single machine, I setup my environment as follows:

1. a Docker container for MongoDB, named `mongo`.
2. a Docker container for MySql, named `mysql`.
3. [Visual Studio Community Edition](https://visualstudio.microsoft.com/vs/community/) for the backend.
4. [NodeJS](https://nodejs.org/en/download/): use the LTS version.
5. [Angular CLI](https://angular.io/cli): this requires NodeJS. Both NodeJS and Angular are used for the frontend.
6. [VSCode](https://code.visualstudio.com/download) to be used as the frontend editor.

Note that you can either install MySql and MongoDB the usual way, or use a Docker-based service. I prefer the latter because it gives you more freedom and does not pollute my workstation with stuff I have then to maintain; yet, you can perfectly go with the former solution, which probably is easier. In this case, just install them and you are ready to go without requiring Docker.

If you are installing your databases rather than using Docker containers for them, just accept all the defaults during setup. For MySql you also need to setup a root user; I usually add an internal root user named `root` with password=`mysql`; these are the defaults in Cadmus. Of course you can change them (in the connection strings); but it's easier if you stick to this convention. This won't hurt anyway, because you will use MySql only in your local machine for development.

## Angular CLI

To install Angular CLI globally:

```bash
npm install -g @angular/cli@latest
```

To upgrade it:

```bash
npm upgrade -g @angular/cli
```

## MongoDB

**Windows**: to run a mongo container persisting data in the host via a volume in `c:\users\<USERNAME>\data\mongo`:

```bash
docker run --name mongo -d -p 27017:27017 --volume c:/users/dfusi/dockerVolMongo/db:/data/db mongo --noauth
```

You can now connect to `localhost`.

Some database clients:

- [Studio3T](https://studio3t.com/)
- [MongoDB Compass](https://www.mongodb.com/products/compass)

## MySql

**Windows**: to run a mysql container storing data in host through a volume in `c:\data\mysql`:

```bash
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=mysql -v c:/data/mysql:/var/lib/mysql -p 3306:3306 mysql --default-authentication-plugin=mysql_native_password
```

Once the container has been created, you can stop and restart it using `docker container stop mysql` and `docker container start mysql`. You can inspect it with `docker container inspect mysql`.

You can now connect to 127.0.0.1 with user=`root` and password=`mysql`.

Some database clients:

- [MySql Workbench](https://dev.mysql.com/downloads/workbench/)
- [DBeaver](https://dbeaver.io/download/)
