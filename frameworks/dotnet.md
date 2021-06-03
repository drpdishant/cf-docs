# Deploying .NET Application 

This guide walks you through deploying your .NET application to Openxcell Cloud.

Following are the configuration files you need to create.

- **manifest.yml** : Cloudfoundry manifest

## manifest.yml

This is file that contains the specifications of the deployment, such as application name, memory, buildpack, and environment variables.

```yaml
---
applications:
- name: my-dotnet-app
  buildpacks:
  - paketo-buildpacks/dotnet-core
  env:
    BP_DOTNET_PROJECT_PATH: ./MyProject/ # Path containing your Project respective to the manifest.ym
    ASPNETCORE_ENVIRONMENT: development # Application Environment
```

## Deploying

Ensure that you are logged into cf cli and have need permission to deploy.

All you need to run is cf push

```bash
cf push
```

**Output**: 

```bash
  ...
  Setting default process type 'web'
  *** Images (sha256:21413a07bdc723591a86abb6c653a7c83394be025145abcb2c4be1bb60422328):
  core.registry.openxcell.dev/cf-for-k8s/4dbdc561-2a90-463d-a41c-5c39368a476d
  core.registry.openxcell.dev/cf-for-k8s/4dbdc561-2a90-463d-a41c-5c39368a476d:b1.20210603.214117
  Adding cache layer 'paketo-buildpacks/dotnet-core-runtime:dotnet-core-runtime'
  Adding cache layer 'paketo-buildpacks/dotnet-core-aspnet:dotnet-core-aspnet'
  Adding cache layer 'paketo-buildpacks/dotnet-core-sdk:dotnet-core-sdk'
  Adding cache layer 'paketo-buildpacks/icu:icu'
   Build successful
  ...
  ...

```

> Output will print url for your Application, for our app `my-java-app` the url will be `my-java-app.apps.openxcell.dev`