# Building Docker Images

- [Building Docker Images](#building-docker-images)
  - [Web API Image](#web-api-image)
  - [Web App Image](#web-app-image)

Quick directions for building images are also found in the readme of each dockerized project.

## Web API Image

Note: currently we cannot rely on private NuGet feeds, like MyGet or Google repository. Thus, the temporary workaround is consuming the required packages from a local folder in the build image. These packages are copied from the local NuGet repository of your machine using a batch.

Thus, to build an image I follow these steps:

1. (applies to my own environment only at this time, as per the mentioned feed workaround): run `UpdateLocalPackages.bat` to update the local packages. Ensure that the version numbers are in synch with the versions used in your projects before running.

2. open a command prompt in the solution folder (where the `Dockerfile` is located) and run a command like this (replace the cadmus image name and version as needed):

```ps1
docker build . -t vedph2020/cadmus_api:1.0.6 -t vedph2020/cadmus_itinera_api:latest
```

If you forget to specify the tag, you can add it later, e.g. `docker tag <imageid> <newtag>`.

3. to publish the image in the Hub: login and push the image:

```ps1
docker login -u vedph
docker push vedph2020/cadmus_api:1.0.6
```

## Web App Image

1. `npm run build-all` to build all the libraries of this app (if any).

2. `ng build --prod` to build the app. Note: after building, you can eventually customize the `env.js` parameters in the `dist` output folder to change parameters like the root web API URI used by the application. This allows you to change these parameters without having to rebuild the application (see [this article](https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/)).

3. run a command like (replace the image name and version as required): `docker build . -t vedph2020/cadmus-itinera-app:1.0.8 -t vedph2020/cadmus-itinera-app:latest`.

4. to publish the image in the Hub, login and push the image like:

```ps1
docker login -u vedph
docker push vedph2020/cadmus-itinera-app:1.0.8
```
