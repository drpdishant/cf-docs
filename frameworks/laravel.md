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
### Application Name and Buildpack

Define the application name and buildpack configurations

```yaml
applications: 
- name: voyager # Unique Application name, will auto-generate the app url e.g voyager.apps.cloud.openxcell.dev
  buildpack: paketo-buildpacks/php #Buildpack to be used for building the application container.
```

### Environment Variables

Define environment variables specific to cloudfoundry and laravel application

```yaml
  env:
    APP_KEY: APP_KEY # Application Key Generated using  php aritisan key:generate --show
    APP_URL: APP_URL # Laravel app url e.g voyager.apps.cloud.openxcell.dev , Should be set according the to app name
    APP_ENV: APP_ENV # Laravel app env, e.g local,development or production
    APP_DEBUG: true # Whether to enable debug or not, Default is false
    CF_STAGING_TIMEOUT: 15 # Max time for staging the app, i.e build,and deploy stages combined. Set to 15 minutes as composer install may take a while.
    CF_STARTUP_TIMEOUT: 15 # Max Time to wait for App Startup once its deployed
    DB_CONNECTION: mysql # Laravel Database connection type
    DB_HOST: my-cluster-mysql-master.default.svc ## Laravel Database URL
    DB_PORT: 3306 # Laravel Database Port
    DB_DATABASE: voyager # Laravel Database name
    DB_USERNAME: voyager # Laravel Database user
    DB_PASSWORD: voyager # Laravel database password
```

### Tasks

Tasks are used to perform one-off jobs, which include:

- Migrating a database
- Sending an email
- Running a batch job
- Running a data processing script
- Processing images
- Optimizing a search index
- Uploading data
- Backing-up data
- Downloading content

```yaml
processes:
  - type: task
    name: migrate
    command: | 
      php artisan migrate --force
      php artisan cache:clear
      php artisan config:clear
      php artisan route:clear
      php artisan optimize
```

Task can be run using the cf cli once the application is deployed.

```bash
cf run-task APP --name TASK_NAME
```

Example:

```bash
cf run-task voyager --name migrate
```

### Final Manifest

The final `manifest.yml` would look something like this

```yaml
---
applications:
- name: voyager
  memory: 100M
  buildpack: paketo-buildpacks/php
  env:
    APP_KEY: base64:Rn7AhcbQMRBr05e4Rpdoy7siwNbG5x0s6tjAqTokHgY=
    APP_URL: https://voyager.apps.cloud.openxcell.dev
    APP_ENV: local
    APP_DEBUG: true
    CF_STAGING_TIMEOUT: 15
    CF_STARTUP_TIMEOUT: 15
    DB_CONNECTION: mysql
    DB_HOST: my-cluster-mysql-master.default.svc
    DB_PORT: 3306
    DB_DATABASE: voyager
    DB_USERNAME: voyager
    DB_PASSWORD: voyager
  processes:
  - type: task
    name: migrate
    command: | 
      php artisan migrate --force
      php artisan cache:clear
      php artisan config:clear
      php artisan route:clear
      php artisan optimize
```

## Writing the `buildpack.yml`

This file contains all the buildpack specific configuration, e.g for php version, composer version, web directory etc.

- Create a file named `buildpack.yml` at project root
### composer configuration

- The composer directive will have configuration for composer such as version, vendor_directory, global packages, etc.

```yaml
composer:
  vendor_directory: vendor
```

### php runtime configuration

- The php directive will have configuration for webserver to use, webdirectory, version to install

```yaml
php:
  # directory where web app code is stored
  # default: htdocs
  webserver: httpd # Web server to use, e.g php-server, httpd, nginx
  webdirectory: public
  version: 7.3.*
```

## Creating `.profile`

This file is a simple bash profile script which will be loaded during the app startup. In our case we are using it as workaround for the php build pack bug related to caching the composer layer

Create a file named `.profile` with below content

```bash
#!/bin/bash
ln -s /workspace/app /layers/paketo-buildpacks_php-composer/php-composer-packages/app
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

```
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