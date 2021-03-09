# Cadmus Hosting Sample

- [Cadmus Hosting Sample](#cadmus-hosting-sample)
  - [Requirements](#requirements)
  - [1. Installing Docker](#1-installing-docker)
  - [2. Docker Compose Script](#2-docker-compose-script)
  - [3. Launching Cadmus](#3-launching-cadmus)
  - [4. MySql Tool](#4-mysql-tool)
  - [5. MongoDB Tool](#5-mongodb-tool)
  - [6. Setup Backup Policy](#6-setup-backup-policy)

In this sample, I outline the main steps in setting up a Cadmus-based solution in a Linux (Ubuntu) VM. In the sample I assume the simplest infrastructure, where all what you have is a Linux machine (usually a VM) connected to the internet.

Please notice that for obvious security reasons it's essential that you **use HTTPS**; so your machine should have some certificate for it.

## Requirements

- Cadmus Docker compose script.
- Docker and Docker compose.
- MySql client tool.
- MongoDB client tool.

The full application stack, from databases up to the web application, is containerized. The database client tools are used to access the database services inside the Docker stack for backup purposes.

## 1. Installing Docker

To install Docker and Docker compose, please follow the [instructions here](https://github.com/vedph/cadmus_doc/blob/master/deploy/docker-setup.md).

## 2. Docker Compose Script

The script should be adapted to the environment by opportunely changing its sensitive parameters.

In my sample, I have first created my script; I'm placing it under `/opt/cadmus`, so:

```bash
cd /opt
mkdir cadmus
cd cadmus
touch docker-compose.yml
```

Then you can edit the empty file with your favorite editor (e.g. `nano docker-compose.yml`) and paste the script you created before.

Alternatively, use FTP to transfer the script. To [connect to FTP using PEM with FileZilla](https://stackoverflow.com/questions/16744863/connect-to-amazon-ec2-file-directory-using-filezilla-and-sftp):

1. preferences/settings/connection/sftp: add key file.
2. locate your PEM file and pick it.
3. add a new site to the site manager with your host VM account user name and address (normal login, SFTP).

As for the **database layers**, nothing changes here. These layer are buried inside the stack, and just require an external volume to ensure data preservation independently from the lifetime of their containers.

The code here looks like this:

```yml
version: "3.7"

services:
  # MongoDB
  cadmus-mongo:
    image: mongo
    container_name: cadmus-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null # --quiet
    ports:
      - 27017:27017
    volumes:
      - /var/db/mongo:/data/db
    networks:
      - cadmus-network

  # MySql
  # https://github.com/docker-library/docs/tree/master/mysql#mysql_database
  # https://docs.docker.com/samples/library/mysql/#environment-variables
  # https://github.com/docker-library/mysql/issues/275 (troubleshooting connection)
  cadmus-index:
    image: mysql
    container_name: cadmus-index
    # https://github.com/docker-library/mysql/issues/454
    command: --default-authentication-plugin=mysql_native_password
    environment:
      # the password that will be set for the MySQL root superuser account
      # Note: use dictionary like here rather than array (- name = value)
      # or you might get MySql connection errors!
      # https://stackoverflow.com/questions/37459031/connecting-to-a-docker-compose-mysql-container-denies-access-but-docker-running/37460872#37460872
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_ROOT_HOST: "%"
    ports:
      - 3306:3306
    volumes:
      - /var/db/mysql:/var/lib/mysql
    networks:
      - cadmus-network
```

While the preceding code can be copied as it is from the corresponding Cadmus solution, most changes instead happen in the **API layer**, under its `environment` section:

1. the connection strings to databases (including Serilog) do not need to be changed, as databases are never accessed from outside the VM host.

2. `SEED_ITEMCOUNT` should be set to `0`, as we do not want mock data in the database.

3. stock users passwords and eventually names should be overridden here. In this sample, I just have 1 stock user, and I'm overriding its password with `STOCKUSERS__0__PASSWORD`, where `0` is the index in the array of stock users.

4. `JWT_SECUREKEY` must be changed to some random key. Just ensure that its length is a multiple of 16.

5. `ALLOWED_ORIGINS` must be set to the IP address of your host VM. This is used for CORS, to allow the web application access the API.

6. `MESSAGING` variables must be set accordingly: API root and app URI to your host VM IP or domain name (followed by `/api/` for `MESSAGING__APIROOTURL`); `MESSAGING__SUPPORTEMAIL` must be the email address dedicated to users support.

7. `MAILER__ISENABLED` must be true to enable mailing; additionally, here I setup my SendGrid credentials: `MAILER__SENDEREMAIL` with the email address of the sender, and `MAILER__APIKEY` with the SendGrid API key.

In this code sample, the fake IP address of the host VM is `100.101.102.103`.

```yml
cadmus-api:
  image: vedph2020/cadmus_tgr_api:1.0.14
  ports:
    # https://stackoverflow.com/questions/48669548/why-does-aspnet-core-start-on-port-80-from-within-docker
    - 59590:80
  depends_on:
    - cadmus-mongo
    - cadmus-index
  # wait for mongo before starting: https://github.com/vishnubob/wait-for-it
  command:
    [
      "./wait-for-it.sh",
      "cadmus-mongo:27017",
      "--",
      "dotnet",
      "CadmusTgrApi.dll",
    ]
  environment:
    # for Windows use : as separator, for non Windows use __
    # (see https://github.com/aspnet/Configuration/issues/469)
    - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-mongo:27017/{0}
    - CONNECTIONSTRINGS__INDEX=Server=cadmus-index;port=3306;Database={0};Uid=root;Pwd=mysql
    - SEED__INDEXDELAY=50
    - SEED__ITEMCOUNT=1
    - SERILOG__CONNECTIONSTRING=mongodb://cadmus-mongo:27017/{0}-log
    - STOCKUSERS__0__PASSWORD=...ThePasswordOfYourStockUser
    - JWT__SECUREKEY=...YourJwtSecureKey
    - ALLOWEDORIGINS__0=http://100.101.102.103
    - MESSAGING__APIROOTURL=http://100.101.102.103/api/
    - MESSAGING__APPROOTURL=http://100.101.102.103/
    - MESSAGING__SUPPORTEMAIL=...TheSupportEmailAddress
    - MAILER__ISENABLED=true
    - MAILER__SENDEREMAIL=...TheSenderEmailAddress
    - MAILER__APIKEY=...TheSendGridApiKey
  networks:
    - cadmus-network
```

Of course, this assumes that our Cadmus solution is using SendGrid to send email messages. Just change the `MESSAGING` parameters accordingly if you are using other services.

Finally, in the **app layer** ensure that the published port number is changed to 80 (from 4200), and that in `cadmus-web` you are using the production version of the app image. The production version is equal to the development version, with the only difference that its `env.js` file has been manually patched before creating the Docker image, in order to set the `apiUrl` variable to your host VM URI, e.g.:

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://100.101.102.103:59590/api/";
  window.__env.databaseId = "cadmus-tgr";
})(this);
```

Thus, the final part of the Docker compose script is equal to the standard one, except for the port number mapping (`80:80` instead of `4200:80`), and the image name (notice the `-prod` suffix here):

```yml
  cadmus-web:
    image: vedph2020/cadmus-tgr-app:1.0.17-prod
    ports:
      - 80:80
    depends_on:
      - cadmus-api
    networks:
      - cadmus-network

networks:
  cadmus-network:
    driver: bridge
```

## 3. Launching Cadmus

Once the script is in place (I placed mine in `/opt/cadmus`), you can **launch Cadmus** by entering the directory where your script is located and typing:

```bash
sudo docker-compose up -d
```

The services will start in a couple of minutes. It is suggested that the first time you launch it _without_ `-d`, so you can see all the log messages in the startup process. Then just break out of it, or open another terminal if you want to keep live logs at hand while Cadmus is running.

Once the service is up and running, **open ports** 443 (or 80 until you have no certificate available for HTTPS) for the web app, and the web API port specified in the Docker compose script (here 59590).

## 4. MySql Tool

Install the MySql tool with this command:

```bash
sudo apt-get update
sudo apt-get install mysql-client
```

You can ensure that MySql is working with `netstat -an | grep 3306`, or `telnet 127.0.0.1 3306`. **Attention**: be sure that the Cadmus containers are running, or you won't get any MySql service to connect to!

You can **connect** to the MySql shell with:

```bash
mysql -u root -p -h 127.0.0.1 -P 3306
```

You can **backup** a database like this:

```bash
mysqldump -h 127.0.0.1 -P 3306 -u root -p'mysql' cadmus-tgr | gzip >./mysql-bak/cadmus-tgr_$(date +\%F).gz
```

## 5. MongoDB Tool

Install the MongoDB client with this command (for Ubuntu; as above, the `apt-get update` is here just as a reminder):

```bash
sudo apt-get update
sudo apt-get install mongo-tools
```

**Attention**: for some reason, the MongoDB tool does not appear to be in synch with the MongoDB image version. This would result in errors when using `mongodump`, unless you add `--forceTableScan` when launching it.

You can **backup** a database like this:

```bash
mongodump --forceTableScan --host="127.0.0.1:27017" --db cadmus-tgr --gzip --archive=./mongo-bak/$(date +\%F).gz
```

## 6. Setup Backup Policy

The backup policy is up to you. To keep things simple, you can just create a `sh` batch file which using the database tools dump your Cadmus databases in some folder, archiving them, and then transfers them to some external storage.

You can then use `cron` to schedule this backup.

For instance, using FTP you could just use CURL like `curl -T thelocalfile ftp://thehost.com --user user:secret`, e.g.:

```bash
curl -T cadmus-tgr_2021-03-06.gz ftp://YourFtpAddress --user YourFtpUser:YourFtpPassword
```
