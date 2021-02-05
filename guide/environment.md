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

You can now connect to `localhost`.

Some database clients:

- [Studio3T](https://studio3t.com/)
- [MongoDB Compass](https://www.mongodb.com/products/compass)

## MySql

**Windows**: to run a mysql container storing data in host through a volume in `c:\data\mysql`:

```ps1
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=mysql -v c:/data/mysql:/var/lib/mysql -p 3306:3306 mysql --default-authentication-plugin=mysql_native_password
```

Once the container has been created, you can stop and restart it using `docker container stop mysql` and `docker container start mysql`. You can inspect it with `docker container inspect mysql`.

You can now connect to 127.0.0.1 with user=`root` and password=`mysql`.

Some database clients:

- [MySql Workbench](https://dev.mysql.com/downloads/workbench/)
- [DBeaver](https://dbeaver.io/download/)
