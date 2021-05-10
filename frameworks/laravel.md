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

<!-- tabs:start -->

### **Windows**

```pwsh
New-Item manifest.yml
```

### **Linux**

```bash
touch manifest.yml
```

### **Mac**

```bash
touch manifest.yml
```

<!-- tabs:end -->

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

## Writing the `buikdpack.yml`

This file contains all the buildpack specific configuration, e.g for php version, composer version, web directory etc.

- Create Buildpack file

<!-- tabs:start -->

### **Windows**

```pwsh
New-Item buildpack.yml
```

### **Linux**

```bash
touch buildpack.yml
```

### **Mac**

```bash
touch buildpack.yml
```

<!-- tabs:end -->

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

```bash
#!/bin/bash
ln -s /workspace/app /layers/paketo-buildpacks_php-composer/php-composer-packages/app
```

## Creating `extensions.ini`

This file contains the list of php extensions to be enabled in the buildpack runtime. It is located at inside .php.ini.d/extensions.ini

<!-- tabs:start -->

### **Windows**

- Create .php.ini.d

```pwsh
mkdir .php.ini.d
```

- Creating extensions.ini

```pwsh
New-Item .\.phpi.ini.d\extensions.ini
```

### **Linux**

- Create .php.ini.d

```bash
mkdir .php.ini.d
```

- Creating extensions.ini

```bash
touch .php.ini.d/extensions.ini
```

### **Mac**

- Create .php.ini.d

```bash
mkdir .php.ini.d
```

- Creating extensions.ini

```bash
touch .php.ini.d/extensions.ini
```

<!-- tabs:end -->

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
