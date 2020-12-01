# Docker Setup

Here are some quick setup hints for Docker.

## Windows

For Windows 10, it is recommended to apply all the latest updates, so that you can take advantage of WSL 2 (Windows Subsystem for Linux). Once your OS is up to date, just follow the directions during Docker setup (you will need to install an additional component when requested).

- download and install Docker Desktop from <https://hub.docker.com/editions/community/docker-ce-desktop-windows> (see <https://docs.docker.com/docker-for-windows/install/>).

## Ubuntu

In the consumer __Linux machine__, you must install both *Docker* and *Docker compose*. To install (see <https://docs.docker.com/install/linux/docker-ce/ubuntu/>):

```bash
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

You can verify that Docker is installed correctly by running the hello-world image:

```bash
sudo docker run hello-world
```

To automatically start Docker:

```bash
sudo systemctl start docker
```

and then:

```bash
sudo systemctl enable docker
```

Check for installation: `docker --version`.

### Ubuntu - Docker-Compose

Install Docker compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.25.0/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

(replace `1.25.0` with the latest docker compose release). Test with `docker-compose --version`.

### Ubuntu - Other Software

Useful apps links for Ubuntu:

- [TeamViewer](https://www.teamviewer.com/en/download/linux/)
- [Chrome](https://www.google.com/intl/en-US/chrome/)
- [Studio 3T](https://studio3t.com/download/): download TAR.GZ, then extract and run `sudo sh extractedfilename.sh`.
- [MongoDB Compass](https://www.mongodb.com/download-center?jmp=nav#compass)
- [MySql Workbench](https://dev.mysql.com/downloads/workbench/): download DEB and install. Root password is `mysql`.
- [DBeaver](https://dbeaver.io/download/): download DEB and install.
- [VSCode](https://code.visualstudio.com/download)
- [NodeJS](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-16-04):

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs
```

Check node version: `node --version`, `npm --version`.

- [Angular CLI](https://tecadmin.net/install-angular-on-ubuntu/): uninstall and reinstall, or just install if no previous version:

```bash
sudo npm uninstall -g @angular/cli
sudo npm cache verify
sudo npm install -g @angular/cli@latest
```

## Mac

- download and install Docker for Desktop from <https://docs.docker.com/docker-for-mac/install/>. As for Windows, it includes docker-compose.
