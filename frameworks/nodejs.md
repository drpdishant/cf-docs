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

## Deploying

- Select target Org and Space and create app

```bash
cf target -o myorg -s myspace
```

```bash
cf create-app mynodejs-app
```

cf cli by default only support setting and updating single environment variable at a time. To update in bulk we'll be using [dotenv-to-json](https://www.npmjs.com/package/dotenv-to-json) to conver our env to json format and cf curl to PATCH the environment variables in bulk on CF API.

- Install `dotenv-to-json`

```bash
npm install -g dotenv-to-json
```

- Convert .env to JSON

Cloudfoundry API accepts environment variables payload in following format, as referenced from its [documentation](https://v3-apidocs.cloudfoundry.org/version/3.101.0/index.html#update-environment-variables-for-an-app)

```json
{
"var": {
    "DEBUG": "false",
    "USER": null
    }
}
```
So we'll be using the dotenv-to-json to and combile bash commands in a pipe to get output in desired format.

```bash
echo {\"var\": > env.json ;cat .env | dotenv-to-json | jq >> env.json;echo } >> env.json
```

> Replace .env with whatever file name you have your environment variables stored in



-  Patching the Environment on CF API

cf curl is a wrapper around curl with cf cli to run api operations with the logged in user's authentication. 

To update the environment variables for an app name we require the app guid which we are directly substituting using `$(cf app app_name --guid)`, our command will look like,

```bash
cf curl "/v3/apps/$(cf app mynodejs-app --guid)/environment_variables" -X PATCH -d @env.json
```

- Time to get it up and running

Once you have patched the environment variables, we are good go with deployment

```
cf push
```

**Output:**

```
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