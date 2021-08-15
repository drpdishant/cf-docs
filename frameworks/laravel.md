# Deploying Laravel Application

This guide walks you through deploying your laravel application to Openxcell Cloud.

Following are the configuration files that will be required:

- manifest.yml - Cloudfoundry manifest
- buildpack.yml - PHP Buildpack Specific Configuration. Here we'll be using [paketo buildpack for php](https://paketo.io/docs/buildpacks/language-family-buildpacks/php/).
- .profile - Symlinks to provide workaround for paketo buildpack related issue with composer layer caching. ()
- extensions.ini - PHP Extensions to be enabled

## Writing the `manifest.yml`

Manifest is a yaml file consisting of necessary configurations such as application name and buildpack to be used. 

Create a file named `manifest.yml` at project root

Define the application name and buildpack configurations

```yaml
applications: 
- name: voyager # Unique Application name, will auto-generate the app url e.g voyager.apps.cloud.openxcell.dev
  buildpack: paketo-buildpacks/php #Buildpack to be used for building the application container.
  env:
    CF_STAGING_TIMEOUT: 15 # Max time for staging the app, i.e build,and deploy stages combined. Set to 15 minutes as composer install may take a while.
    CF_STARTUP_TIMEOUT: 15 # Max Time to wait for App Startup once its deployed
```

## Writing the `buildpack.yml`

This file contains all the buildpack specific configuration, e.g for php version, composer version, web directory etc.

- Create a file named `buildpack.yml` at project root, insert the following content into it.
### php runtime configuration

- The php directive will have configuration for webserver to use, webdirectory, version to install

```yaml
php:
  # directory where web app code is stored
  # default: htdocs
  webserver: httpd # Web server to use, e.g php-server, httpd, nginx
  webdirectory: public
  version: 7.4.* #PHP Version to use, Default is 8.0 or as defined in composer.json
```

When using php runtime you get 3 different options to choose for webserver, of which we will be exploring configuration options for **httpd(apache)** and **nginx** webservers

### webserver: httpd

If you choose **httpd** as your webserver you need to place .htacess file into `public` directory of your laravel project with following content.

```bash
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
### webserver: nginx

If you choose **httpd** as your webserver you need create a directory named .nginx.conf.d into your project root, inside that create a file named extras-server.conf with the following content

```bash
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

## Creating `.profile`

This file is a simple bash profile script which will be loaded during the app startup. In our case we are using it as workaround for the php build pack bug related to caching the composer layer

Create a file named `.profile` with below content

```bash
#!/bin/bash
ln -s /workspace/app /layers/paketo-buildpacks_php-composer/php-composer-packages/app
ln -s /layers/paketo-buildpacks_php-composer/php-composer-packages/vendor /workspace/vendor
```

## Creating `extensions.ini`

This file contains the list of php extensions to be enabled in the buildpack runtime. It is located at inside .php.ini.d/extensions.ini

Create a file named `extensions.ini` inside .php.ini.d directory at project root

- Write list of enabled extensions as shown below

```ini
extension=openssl.so
extension=pdo.so
extension=pdo_mysql.so
extension=mbstring.so
extension=bz2.so
extension=zlib.so
```

## [Setting up environment variables](./operations/setenv.md)
## And .... Deploy


Ensure that cf cli is installed and configured on your system and you are logged in to the api.

- Check

```bash
cf target
```

Output:

```bash
API endpoint:   https://api.cloud.openxcell.dev
API version:    3.100.0
user:           testing
org:            openxcell
space:          test
```

- Push

```bash
cf push
```
Output:

```bash
   Build successful

Waiting for app voyager to start...

Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...

name:                voyager
requested state:     started
isolation segment:   placeholder
routes:              voyager.apps.cloud.openxcell.dev
last uploaded:       Mon 10 May 00:55:35 IST 2021
stack:
buildpacks:
isolation segment:   placeholder
```

The application url is printed in value for routes use it to access your application.

You can check the app url by getting the app details

```bash
cf app voyager
```

Output:

```bash
Showing health and status for app voyager in org openxcell / space test as testing...

name:                voyager
requested state:     started
isolation segment:   placeholder
routes:              voyager.apps.cloud.openxcell.dev
last uploaded:       Mon 10 May 00:55:35 IST 2021
stack:
buildpacks:
isolation segment:   placeholder

type:           web
sidecars:
instances:      1/1
memory usage:   100M
     state     since                  cpu    memory       disk       details
#0   running   2021-05-09T19:26:24Z   0.0%   13.3M of 0   60K of 0   

type:           task
sidecars:
instances:      0/0
memory usage:   1024M
There are no running instances of this process.
```

### Running Tasks

Once the app is deployed we can run the one-off task defined in manifest.

- Run task

```bash
cf run-task voyager
```

- Check status

```bash
cf tasks voyager
```

Output:

```bash
Getting tasks for app voyager in org openxcell / space test as admin...

id   name      state       start time                      command
1    migrate   SUCCEEDED   Mon, 10 May 2021 16:46:30 UTC   php artisan migrate --force
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan optimize
php artisan storage:link
```

- Check logs

```bash
cf logs voyager --recent
```

## Creating and Binding Shared MySQL Service

Shared MySQL Service is built and deployed with the [shared-mysql-service-broker](https://github.com/making/shared-mysql-service-broker).
Its configured with an existing database and provides and open service broker interface for provisioning database as service and binding credentials with application

You can list the service in marketplace but running,

```bash
cf marketplace
```

Output:

```bash
Getting all service offerings from marketplace in org openxcell / space test as testing...

offering       plans    description    broker
shared-mysql   shared   Shared MySQL   shared-mysql
```

### Creating Database (Creating Service Instance)

Creating the service instance for the shared-mysql service will create a database onto the shared db server and make it available for binding with application

```
cf create-service shared-mysql shared voyager-db
```

### Binding the Service

Binding the service instance to an application will insert the service info and credentials into the environment of the application, which will then be dynamically read by the application to connected to the service.
Binding a service instance to an application inserts the details into an environment variable named VCAP_SERVICE

- Bind

```bash
cf bind-service voyager-db voyager
```

Binding a service instance requires applicated to be restaged in order to update the environment variables

- Restage

```bash
cf restage voyager
```

Once restage is complete you can check the env and look for VCAP_SERVICE

```bash
cf env voyager
```

```json
VCAP_SERVICES: {
 "shared-mysql": [
  {
   "binding_guid": "7f96d75b-b707-4af6-b1bb-ff28ef2b37cb",
   "binding_name": null,
   "credentials": {
    "hostname": "my-cluster-mysql-master.default.svc",
    "jdbcUrl": "jdbc:mysql://my-cluster-mysql-master.default.svc:3306/cf_4122847937834fd7b9f3c39724a4b707?useSSL=false\u0026allowPublicKeyRetrieval=true\u0026user=9af51e13c99b849d\u0026password=da51dd65adeb4a108be432e47af7b1aa",
    "name": "cf_4122847937834fd7b9f3c39724a4b707",
    "password": "da51dd65adeb4a108be432e47af7b1aa",
    "port": 3306,
    "uri": "mysql://9af51e13c99b849d:da51dd65adeb4a108be432e47af7b1aa@my-cluster-mysql-master.default.svc:3306/cf_4122847937834fd7b9f3c39724a4b707?useSSL=false\u0026allowPublicKeyRetrieval=true\u0026reconnect=true",
    "username": "9af51e13c99b849d"
   },
   "instance_guid": "e4a8bc58-d19f-4644-8d3c-b6561a8e7027",
   "instance_name": "voyager",
   "label": "shared-mysql",
   "name": "voyager",
   "plan": "shared",
   "provider": null,
   "syslog_drain_url": null,
   "tags": [
    "mysql"
   ],
   "volume_mounts": []
  }
 ]
}
```

> VCAP SQUASH

> To be able to use the service credentials directly as environment variables you can use [vcap-squash](https://github.com/cloudfoundry-community/vcap-squash), a tool that squashes VCAP_SERVICE credentials as flat environment variables. You can check out the usage on its [github page](https://github.com/cloudfoundry-community/vcap-squash).

- Download and place its binary in project root.

```bash
curl -Lo vcap-squash https://github.com/cloudfoundry-community/vcap-squash/releases/download/v0.1.1/vcap-squash-linux-amd64
```

- Update `.profile` as shown

```bash
#!/bin/bash
ln -s /workspace/app /layers/paketo-buildpacks_php-composer/php-composer-packages/app
ln -s /layers/paketo-buildpacks_php-composer/php-composer-packages/vendor /workspace/vendor
eval $(./vcap-squash)
```

- Now the environment variables will be available as shown

| JSON KEY | ENV |
| -- | -- |
| shared-mysql.credentials.hostname | SHARED_MYSQL_HOSTNAME |
| shared-mysql.credentials.username | SHARED_MYSQL_USERNAME |
| shared-mysql.credentials.name | SHARED_MYSQL_NAME |
| shared-mysql.credentials.password | SHARED_MYSQL_PASSWORD |
| shared-mysql.credentials.port | SHARED_MYSQL_PORT |

- Editing the manifest to use the above variables.

```yaml
applications:
- name: voyager
  memory: 100M
  buildpack: paketo-buildpacks/php
  env:
    ...
    ...
    DB_CONNECTION: mysql
    DB_HOST: ${SHARED_MYSQL_HOSTNAME}
    DB_PORT: ${SHARED_MYSQL_PORT}
    DB_DATABASE: ${SHARED_MYSQL_NAME}
    DB_USERNAME: ${SHARED_MYSQL_USERNAME}
    DB_PASSWORD: ${SHARED_MYSQL_PASSWORD}
  ...
```