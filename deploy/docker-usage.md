# Docker: Using Images

This section contains practical instructions for using the Cadmus Docker images. You can use Windows, Linux, or Mac. For Linux the suggested distribution is Ubuntu.

We assume that you already have **Docker** installed and working (see [here](./docker-setup.md)). For Windows 10, it is recommended to apply all the latest updates, so that you can take advantage of WSL 2 (Windows Subsystem for Linux). Once your OS is up to date, just follow the directions during Docker setup (you will need to install an additional component when requested).

## Image cadmus_api

This uses a Docker stack for all the backend services, while directly running the web application from your own machine via NodeJS. This is the recommended approach for faster testing the web app, as it does not require me to rebuild and upload its image for every single change (at this stage, rebuilding takes a relevant amount of machine time): you just have to update your code from the repository (using GitHub, or just redownloading it as a whole ZIP), whenever I update it.

To this end, you need to install some additional tools:

1. **NodeJS**: use the [installer](https://nodejs.org/en/download/) for Windows/Mac; `apt install npm` in Linux.
2. **Angular 10+**: once NodeJS has been installed, run this in a command prompt: `npm install -g @angular/cli@latest`.
3. **Nrwl/nx**: once NodeJS has been installed, run this in a command prompt: `npm install -g @nrwl/cli`.

### Running Cadmus

Follow these directions:

1. save the API `docker-compose.yml` file (from <https://github.com/vedph/cadmus_api>) somewhere in your machine. **Note** that you should not right-click and download, as this downloads the GitHub HTML page including that code, and not the raw code. An easy way to download the code is clicking the `docker-compose.yml` file in GitHub, thus opening it, and then pick `Raw` at the top-right. When code only gets displayed, just save it somewhere with name `docker-compose.yml`.

2. from a command prompt, enter the directory where you saved the `docker-compose.yml` file, and type the command `docker-compose up` (use `sudo` for Linux). This fires the backend services, and the first time it creates and seeds mock databases; so this will take a couple of minutes. You will see a number of log information being dumped on the screen, and a total of 100 items being seeded into the newly created database. You should wait until the dump stops with no more messages. Please note that to allow other services startup, the API waits for 25 seconds before continuing; so, depending on the speed of your host system, it might happen that the other services are already done and the API is still waiting, and in this interval of time you won't see any message dumped; it will resume as soon as the API continues.

3. if everything went OK, open your browser at `localhost:60380/swagger`. You will see the current Cadmus API surface.

Now the backend service is running, so you are ready to start the web application. When you have finished and want to stop the service, break from it in the console (CTRL+C or the like); to clean up everything, enter `sudo docker-compose down`. This will remove all the data together with the container, so you can start fresh again.

4. download (ZIP) or clone the [web application Git repository](https://github.com/vedph/cadmus_web). You should pull from this repository or redownload it whenever it gets updated.

5. open a command prompt in the root folder of the cloned/downloaded repository, and enter these commands (in Linux for `npm` use `sudo`):

```bash
npm i
nx serve
```

The first command restores the dependencies packages, and will take some minutes; the first time it will download a lot of code; the next times, it will just ensure that all the required dependencies are there, downloading something if it is missing.

The second command starts the Angular app. It will dump some messages on the screen, and finally stop telling you that you can open your browser at localhost:4200.

6. open your browser at `localhost:4200`. Login with the following (fake) credentials:

- username: `zeus`
- password: `P4ss-W0rd!`

These credentials are found in the `appsettings.json` configuration file of this project. Please notice that in production they are always replaced with true credentials, set in server environment variables.

Please use an evergreen (modern) and up-to-date browser like Chrome, Firefox, or Edge.

### Connecting to Databases (Advanced)

This is for more advanced users. Should you want to check the underlying databases directly, you can connect to them through the Docker container.

There are two databases, one for the data proper (MongoDB), another for their index, used in real-time searches during editing (MySql).

To connect to the MongoDB database from the Docker host, using e.g. [Compass](https://www.mongodb.com/download-center?jmp=nav#compass>) or [Studio 3T](https://studio3t.com/download/):

- server: 127.0.0.1
- port: 27017 (the default)
- no authentication

To connect to the MySql database from the Docker host, using e.g. [MySql Workbench](https://dev.mysql.com/downloads/workbench/):

- server: localhost
- port: 3306 (the default)
- user: `root`
- password: `mysql` (mind the case: all lowercase)

Note: on some systems, MySql Workbench setup might fail with an error telling you that some Visual Studio files are missing. In this case, stop the installation, visit [this page](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads), and download the appropriate VC redistributable file (e.g. for Windows this typically is [vc_redist.x64.exe](https://aka.ms/vs/16/release/vc_redist.x64.exe)).

### Image cadmus_web

This image and its compose script are used to start a container providing the full backend and frontend stacks (databases, API, web application). This image thus provides the full-stack system, ready to be used with a mock database, without the hassle of setting up Angular and its dependencies. The disadvantage is that we need to build and publish this image whenever the frontend is changed, and the frontend is the fastest-moving part of the system.

The directions are similar to the above:

1. save the API `docker-compose.yml` file (from <https://github.com/vedph/cadmus_web>) somewhere in your machine. Please notice that this file comes from another repository, even if it's still named `docker-compose.yml`. You should place each in its own, distinct directory.

2. from a command prompt, enter the directory where you saved the `docker-compose.yml` file, and type the command `docker-compose up` (use `sudo` for Linux). This fires all the services, backend and frontend, so please refer to the instructions above for more. As explained above, you can use `docker-compose down` to remove the containers and restart fresh, with a new database.

3. if everything went OK, open your browser at `localhost:4200`. To login:

- username: `zeus`
- password: `P4ss-W0rd!`

These credentials are found in the `appsettings.json` configuration file of the [API project](https://github.com/vedph/cadmus_api). Please notice that in production they are always replaced with true credentials, set in server environment variables.
