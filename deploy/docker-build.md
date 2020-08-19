# Building Docker Images

The images which can be built are:

- `cadmus_api` for the backend;
- `cadmus_web` for the frontend.

## Image cadmus_api

Note: currently we cannot rely on private NuGet feeds, like MyGet or Google repository. Thus, the temporary workaround is consuming the required packages from a local folder in the build image. These packages are copied from the local NuGet repository of your machine using a batch.

Thus, to build an image I follow these steps:

1. (applies to my own environment only at this time, as per the mentioned feed workaround): run `UpdateLocalPackages.bat` to update the local packages. Ensure that the version numbers are in synch with the versions used in your projects before running.

2. open a command prompt in the solution folder (where the `Dockerfile` is located) and run:

```ps1
docker build . -t vedph2020/cadmus_api:latest
```

If you forget to specify the tag, you can add it later, e.g. `docker tag <imageid> <newtag>`.

3. to publish the image in the Hub: login and push the image:

```ps1
docker login --username vedph
docker push vedph2020/cadmus_api:latest
```

## Image cadmus_web

The Docker Hub for the frontend is configured to automatically build on push. Should you need to manually build an image:

1. build the app. After building, you can eventually customize the `env.js` parameters in the `dist` output folder.

```ps1
nx build --prod
```

The `env.js` file is an "environment" settings file you can use to configure essential environment variables without having to rebuild the application (see [this article](https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/)). All what you have to do is editing values inside that file, once the application has been built. Currently, these values are defined:

- `apiUrl`: the root URL to the API: `http://localhost:60304/api/`;
- `databaseId`: the name of the Cadmus database: `cadmus`;
- `name`: the title of the web application: `Demo`.

1. build the image:

```ps1
docker build . -t vedph2020/cadmus_web:latest
```

3. to publish the image in the Hub, login and push the image:

```ps1
docker login --username vedph
docker push vedph2020/cadmus_web:latest
```
