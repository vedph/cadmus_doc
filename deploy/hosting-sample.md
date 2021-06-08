# Cadmus Hosting Sample

- [Cadmus Hosting Sample](#cadmus-hosting-sample)
  - [Requirements](#requirements)
  - [1. Installing Docker](#1-installing-docker)
  - [2. Docker Compose Script](#2-docker-compose-script)
  - [3. Launching Cadmus](#3-launching-cadmus)
  - [4. MySql Tool](#4-mysql-tool)
  - [5. MongoDB Tool](#5-mongodb-tool)
  - [6. Setup Backup Policy](#6-setup-backup-policy)
  - [Setting up HTTPS](#setting-up-https)
    - [Getting a Certificate](#getting-a-certificate)
    - [Passing Certificates to the API](#passing-certificates-to-the-api)
    - [Passing Certificates to the Frontend](#passing-certificates-to-the-frontend)
    - [Recap](#recap)

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

Alternatively, use SCP or FTP to transfer the script. To [connect to FTP using PEM with FileZilla](https://stackoverflow.com/questions/16744863/connect-to-amazon-ec2-file-directory-using-filezilla-and-sftp):

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
    # https://stackoverflow.com/questions/55559386/how-to-fix-mbind-operation-not-permitted-in-mysql-error-log
    cap_add:
      - SYS_NICE   
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

2. `SEED__ITEMCOUNT` should be set to `0`, as we do not want mock data in the database.

3. stock users passwords and eventually names should be overridden here. In this sample, I just have 1 stock user, and I'm overriding its password with `STOCKUSERS__0__PASSWORD`, where `0` is the index in the array of stock users.

4. `JWT__SECUREKEY` must be changed to some random key. Just ensure that its length is a multiple of 16.

5. `ALLOWED__ORIGINS` must be set to the IP address of your host VM. This is used for CORS, to allow the web application access the API.

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

Finally, in the **app layer** ensure that the published port number is changed to 80 (from 4200; or 443 for SSL, but usually we first setup for HTTP to test the stack, and then add HTTPS); and that in `cadmus-web` you are using the *production version* of the app image.

The production version of the web app is equal to the development version, with the only difference that its `env.js` file has been manually patched before creating the Docker image, in order to set the `apiUrl` variable to your host VM URI, e.g.:

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

## Setting up HTTPS

To use HTTPS, you need to make some changes to the previous configuration.The main steps are outlined here.

### Getting a Certificate

Of course, first you need a certificate. Get it from somewhere, and prepare it in these formats:

- CER with KEY
- PFX: you can convert a CER (or other format) certificate with its key file into a PFX file using the Linux `openssl` tool, like this:

```bash
openssl pkcs12 -export -out certificate.pfx -inkey <NAME>.key -in <NAME>.cer
```

Here your mileage may vary according to the format you get for the certificate. See the [openssl manual](https://www.openssl.org/docs/manmaster/man1/) for more, e.g. to convert PEM into CRT and KEY:

```bash
openssl x509 -outform der -in fullchain.pem -out docker_it.crt
openssl rsa -outform der -in privkey.pem -out docker_it.key
```

Then place the 3 files under some folder in your host, e.g.:

- my CER and KEY files are under `opt/cadmus/cert`.
- my PFX file is under `opt/cadmus/nginx`.

This is required because we are going to share these certificates with the services running inside the Cadmus Docker containers via a Docker volume. This is the suggested way of making a certificate available to what's inside a Docker image, without having to build an image with the certificate inside it, which would not be secure nor practical.

### Passing Certificates to the API

- reference: [ASP.NET CORE SSL in Docker](https://docs.microsoft.com/en-us/aspnet/core/security/docker-compose-https?view=aspnetcore-5.0).

Next step is passing our certificates to the API service inside the Cadmus Docker stack.

To this end, in the API service of the Docker compose script add a volume to share the PFX certificate file:

```yml
volumes:
  - /opt/cadmus/cert/certificate.pfx:/app/Infrastructure/Certificate/certificate.pfx
```

This instruction means that the file `certificate.pfx` under folder `/opt/cadmus/cert` in our host machine will be shared via a Docker volume pointing to file `/app/Infrastructure/Certificate/certificate.pfx` in the Docker image file system. This location is where Kestrel, which is the web server used by ASP.NET Core Cadmus API are built with, will be instructed to find the certificate.

Then, add these environment variables in the same API service section of the Docker script to tell Kestrel about the certificate:

```yml
environment:
  - ASPNETCORE_URLS=https://+:443;http://+:80
  - ASPNETCORE_Kestrel__Certificates__Default__Path=/app/Infrastructure/Certificate/certificate.pfx
  - ASPNETCORE_Kestrel__Certificates__Default__Password=YOURCERTPASSWORD
```

These variables tell Kestrel that the HTTPS port is 443, the HTTP port is 80, the certificate is found at the specified path, and its password is that specified.

**NOTE**: be sure to mind the number of underscore (`_`) characters in these variable names! Please note that `ASPNETCORE_` ends with 1 underscore, whereas all the other underscores in the variable name come with 2 of them.

Also, still in this `environment` section, be sure to add an HTTPS (rather than HTTP) allowed origin for CORS, e.g.:

```yml
  - ALLOWEDORIGINS__1=https://docker.somewhere.it
```

Please notice that `__1` here is just the index in an array of allowed origins. So here it happens to be `1` because it's the second origin I put in my list. Change this index accordingly, remember that it's 0-based (so the first origin you add is `ALLOWEDORIGINS__0`, the second `ALLOWEDORIGINS__1`, etc.).

So, this is enough for the API service: all what we did was sharing our PFX certificate file with Kestrel running in the Docker container, and telling it where to find this certificate.

Also, given that our frontend web app will be served in HTTPS, we ensured that its URI is among the allowed origins for this API.

### Passing Certificates to the Frontend

The frontend part uses NGINX to serve the web app. You can find the default NGINX configuration for it among the source files of each Cadmus web app, in its GitHub repository.

(1) Rather than modifying this configuration and rebuild the image, we're going to replace it with a new one, using our CER and KEY certificates.

The new NGINX configuration replaces the original `server` section with this one:

```nginx
server {
  listen 443 ssl;
  listen 80;
  server_name localhost;
  ssl_certificate docker_it.cer;
  ssl_certificate_key docker_it.key;

  gzip on;
  gzip_min_length 1000;
  gzip_proxied expired no-cache no-store private auth;
  gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
}
```

The relevant changes are:

- `listen 443 ssl` for HTTPS.
- the `ssl_certificate` lines to define the CER and KEY certificates to be used.

The rest of the section is unchanged. Save this new `nginx.conf` configuration file somewhere in your host, e.g. in `/opt/cadmus/nginx`.

(2) Then, in the Docker compose script, head to the `cadmus-web` service section, and make these changes:

- ensure you have changed the image name to reference your production version for the app. The default image name in the GitHub source refers to the development version, which targets API in localhost. Your production version should change the `env.js` file so that API are located at their HTTPS URIs.
- ensure that both ports 80 and 443 are exposed.
- add volumes to pass the NGINX configuration and the CER and KEY files to the Docker container.

Here is a sample:

```yml
cadmus-web:
  restart: always
  image: vedph2020/cadmus-itinera-app:1.0.35-prod
  ports:
    - 80:80
    - 443:443
  volumes:
    # https://www.techrepublic.com/article/how-to-enable-ssl-on-nginx/
    # overwrite nginx.conf to use SSL with certificates from the host
    - /opt/cadmus/nginx/nginx.conf:/etc/nginx/nginx.conf
    - /opt/cadmus/nginx/docker_it.cer:/etc/nginx/docker_it.cer
    - /opt/cadmus/nginx/docker_it.key:/etc/nginx/docker_it.key
  depends_on:
    - cadmus-api
```

### Recap

As a recap, this is the full Docker compose for HTTPS using as a sample a version of the Itinera project (of course, all the sensible data were removed):

```yml
version: '3.7'

services:
  # MongoDB
  cadmus-mongo:
    restart: always
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
  cadmus-mysql:
    restart: always
    image: mysql
    container_name: cadmus-mysql
    # https://stackoverflow.com/questions/55559386/how-to-fix-mbind-operation-not-permitted-in-mysql-error-log
    cap_add:
      - SYS_NICE   
    # https://github.com/docker-library/mysql/issues/454
    command: --default-authentication-plugin=mysql_native_password
    environment:
      # the password that will be set for the MySQL root superuser account
      # Note: use dictionary like here rather than array (- name = value)
      # or you might get MySql connection errors!
      # https://stackoverflow.com/questions/37459031/connecting-to-a-docker-compose-mysql-container-denies-access-but-docker-running/37460872#37460872
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_ROOT_HOST: '%'
    ports:
      - 3306:3306
    volumes:
      - /var/db/mysql:/var/lib/mysql
    networks:
      - cadmus-network

  cadmus-api:
    restart: always
    image: vedph2020/cadmus_itinera_api:1.0.33
    ports:
      # https://stackoverflow.com/questions/48669548/why-does-aspnet-core-start-on-port-80-from-within-docker
      - 54183:80
      - 54184:443
    depends_on:
      - cadmus-mongo
      - cadmus-mysql
    # wait for mongo before starting: https://github.com/vishnubob/wait-for-it
    command: ["./wait-for-it.sh", "cadmus-mongo:27017", "--", "dotnet", "CadmusItineraApi.dll"]
    volumes:
      - /opt/cadmus/cert/certificate.pfx:/app/Infrastructure/Certificate/certificate.pfx
    environment:
      # for Windows use : as separator, for non Windows use __
      # (see https://github.com/aspnet/Configuration/issues/469)
      # https://docs.microsoft.com/en-us/aspnet/core/security/docker-compose-https?view=aspnetcore-5.0
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/app/Infrastructure/Certificate/certificate.pfx
      - ASPNETCORE_Kestrel__Certificates__Default__Password=<CERTPASS>
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-mongo:27017/{0}
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-mysql;port=3306;Database={0};Uid=root;Pwd=mysql
      - SEED__ITEMCOUNT=1
      - SEED__INDEXDELAY=50
      - MESSAGING__APIROOTURL=https://itinera.somewhere.com
      - MESSAGING__APPROOTURL=https://itinera.somewhere.it/
      - MESSAGING__SUPPORTEMAIL=<SUPPORTEMAIL>
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=<YOURPASSWORDHERE>
      - JWT__SECUREKEY=<YOURKEYHERE>
      - ALLOWEDORIGINS__0=https://yourdomain.com
      - MAILER__ISENABLED=true
      - MAILER__SENDEREMAIL=<EMAIL>
      - MAILER__SENDERNAME=<SENDERNAME>
      - MAILER__HOST=<SMTPADDRESS>
      - MAILER__PORT=25
      - MAILER__USESSL=false
    networks:
      - cadmus-network

  cadmus-biblio-api:
    restart: always
    image: vedph2020/cadmus_biblio_api:1.0.9
    ports:
      - 61691:80
      - 61692:443
    depends_on:
      - cadmus-mongo
      - cadmus-mysql
    volumes:
      - /opt/cadmus/cert/certificate.pfx:/app/Infrastructure/Certificate/certificate.pfx
    environment:
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/app/Infrastructure/Certificate/certificate.pfx
      - ASPNETCORE_Kestrel__Certificates__Default__Password=<CERTPASS>
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-mongo:27017/{0}
      - CONNECTIONSTRINGS__BIBLIO=Server=cadmus-mysql;port=3306;Database={0};Uid=root;Pwd=mysql
      - SEED__BIBLIODELAY=50
      - SEED__ENTITYCOUNT=1
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=<YOURPASSWORD>
      - ALLOWEDORIGINS__0=https://itinera.somewhere.it
    networks:
      - cadmus-network

  cadmus-web:
    restart: always
    image: vedph2020/cadmus-itinera-app:1.0.35-prod
    ports:
      - 80:80
      - 443:443
    volumes:
      # https://www.techrepublic.com/article/how-to-enable-ssl-on-nginx/
      # overwrite nginx.conf to use SSL with certificates from the host
      - /opt/cadmus/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /opt/cadmus/nginx/docker_it.cer:/etc/nginx/docker_it.cer
      - /opt/cadmus/nginx/docker_it.key:/etc/nginx/docker_it.key
    depends_on:
      - cadmus-api
```
