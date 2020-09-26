# Quick Start

This quick start will lead you to setup an early version of the Cadmus prototype on your own system. Please notice that currently this is just an early UI preview under active development, so that this procedure is used for testing purposes only.

The fastest way to get playing with Cadmus is using Docker, even if this is not a requirement, and you could also manually setup your services.

1. ensure you have installed [Docker](https://docs.docker.com/engine/install/) on your system. You must install also Docker compose; this is already installed when using an installer (e.g. in Windows), while it requires to be installed via command line in Linux (see below).

2. download the [Docker compose script](https://github.com/vedph/cadmus_web/blob/master/docker-compose.yml), saving it anywhere in your system.

3. open a terminal window, and enter the directory where you downloaded the script. Then type this command (be sure to prefix it with `sudo` for Linux-based platforms):

```bash
docker-compose up
```

This should start the underlying MongoDB and MySql services, and the Cadmus API layer, which will display a number of diagnostic messages to the terminal; you should see a mock database being created and seeded with 100 items. Wait until the messages stop with no errors.

4. open a web browser and navigate to `http://localhost:4200` for the Cadmus web application. You could also navigate to `http://localhost:60380/swagger` to see the API. Login with these credentials:

- username: `zeus`
- password: `P4ss-W0rd!`

## Installing Docker Compose

In Linux (see <https://docs.docker.com/compose/install/#install-compose>), you can use these commands (in the first of the following commands, use the latest version number; you can get it from <https://github.com/docker/compose/releases>):

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

sudo curl -L https://raw.githubusercontent.com/docker/compose/1.21.2/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```
