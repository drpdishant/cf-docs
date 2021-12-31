# Deploying NodeJS Application

This guide walks you through deploying your NodeJS application to Openxcell Cloud.

Following are the configuration files you need to create.

- **manifest.yml** : Cloudfoundry manifest
- **.cfignore** : list of paths and files to ignore during cf push


## manifest.yml

This is file that contains the specifications of the deployment, such as application name, memory, buildpack, and environment variables.

```yaml
---
applications:
 - name: mynodejs-app
   memory: 1g
   buildpack: paketo-buildpacks/nodejs
   env: 
    NODE_ENV: development #Node Env
    BP_NODE_VERSION: "12.x" #To Define Node Version Explicitly
```

Deployment uses paketo buildpacks which will detect the builder, build the application and deploy.

For more buildpack specific cofiguration refer documentation for [paketo java buildpack](https://paketo.io/docs/buildpacks/language-family-buildpacks/nodejs/)

## .cfignore

```txt
node_modules
```

## HealthCheck

To setup health check endpoints for express server, you can use [Express Actuator](https://www.npmjs.com/package/express-actuator) Package.

**Example:**

```bash
const actuator = require('express-actuator');
const app = express();

app.use(actuator());
```

## Deploying

- Select target Org and Space and create app

```bash
cf target -o myorg -s myspace
```

```bash
cf create-app mynodejs-app
```

- [Set/Update the Runtime environment Variables](./operations/setenv)

Follow the linked documentation for how to set/update environment variables for application on cloudfoundry.

- CF Push

Once you have set the environment variables, we are good go with deployment

```bash
cf push
```

**Output:**

```bash
name:                mynodejs-app
requested state:     started
isolation segment:   placeholder
routes:              mynodejs-app.apps.openxcell.dev
last uploaded:       Tue 22 Jun 18:32:28 IST 2021
stack:               
buildpacks:          
isolation segment:   placeholder

type:            web
sidecars:        
instances:       1/1
memory usage:    1024M
start command:   node server.js
     state     since                  cpu    memory       disk        details
#0   running   2021-06-22T13:03:50Z   0.0%   52.2M of 0   1.1M of 0
```

> Output will print url for your Application, for our app `mynodejs-app` the url will be `mynodejs-app.apps.openxcell.dev`