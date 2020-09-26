# Docker: Using Images

This section contains practical instructions for using the Cadmus Docker images.

## Quick Start

Assuming that you already have **Docker** installed, to use Cadmus you can follow these steps (which are equally valid for both Linux and Windows; of course, for Windows you should omit the `sudo` prefix where found):

1. save the `docker-compose.yml` file somewhere in your machine.

2. login into Docker using your Docker account: `sudo docker login --username <DOCKERUSERNAME>`. You will be prompted for the password. This is required if you need to download images.

3. from a command prompt, enter the directory where you saved the `docker-compose.yml` file, and type the command `sudo docker-compose up`.

4. when you want to stop the service, break from it in the console; to clean up everything enter `sudo docker-compose down`.

### Image cadmus_api

This image and its compose script are used to start a container providing the full backend stack (databases and API).

When firing this compose, if everything went OK open your browser at `localhost:60380/swagger`. You will see the current Cadmus API surface.

To *connect to MongoDB databases* from the Linux Docker host, using e.g. Compass (<https://www.mongodb.com/download-center?jmp=nav#compass>):

- server: 127.0.0.1
- port: 27017
- no authentication

You might want to run the frontend web application in your own host machine and let it use the backend from the Docker container:

1. ensure you have installed:

- **NodeJS**
- **Angular 10+**

2. download or clone the web application Git repository (`cadmus_web`).

3. open a command prompt in the root folder of the cloned/downloaded repository, and enter these commands:

```bash
npm i
nx serve
```

The first command restores the dependencies packages and will take some minutes; of course, it should be executed just once. The second command starts the Angular app.

4. open your browser at `localhost:4200`. Login with the following (fake) credentials:

- username: `zeus`
- password: `P4ss-W0rd!`

These credentials are found in the `appsettings.json` configuration file of this project. Please notice that in production they are always replaced with true credentials, set in server environment variables.

### Image cadmus_web

This image and its compose script are used to start a container providing the full backend and frontend stacks (databases, API, web application).

Once you have fired the compose script, open your browser at `localhost:4200`. To login:

- username: `zeus`
- password: `P4ss-W0rd!`

These credentials are found in the `appsettings.json` configuration file of the [API project](https://github.com/vedph/cadmus_api). Please notice that in production they are always replaced with true credentials, set in server environment variables.
