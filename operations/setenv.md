# Setting up Runtime environment variables 

cf cli by default only support setting and updating single environment variable at a time. To update in bulk we'll be using [dotenv-to-json](https://www.npmjs.com/package/dotenv-to-json) to convert our env to json format and cf curl to PATCH the environment variables in bulk on CF API.

## Install `dotenv-to-json`

```bash
npm install -g dotenv-to-json
```
##  Convert .env to JSON

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



## Patching the Environment on CF API

cf curl is a wrapper around curl with cf cli to run api operations with the logged in user's authentication. 

To update the environment variables for an app name we require the app guid which we are directly substituting using `$(cf app app_name --guid)`, our command will look like,

```bash
cf curl "/v3/apps/$(cf app mynodejs-app --guid)/environment_variables" -X PATCH -d @env.json
```

- Time to get it up and running

Once you have patched the environment variables, we are good go with deployment