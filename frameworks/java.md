# Deploying Java Application 

This guide walks you through deploying your Java application to Openxcell Cloud.

Following are the configuration files you need to create.

- **manifest.yml** : Cloudfoundry manifest
- **.cfignore** : list of paths and files to ignore during cf push

## manifest.yml

This is file that contains the specifications of the deployment, such as application name, memory, buildpack, and environment variables.

```yaml
---
applications:
- name: my-java-app
  memory: 1g
  buildpacks:
   - paketo-buildpacks/java
  env:
    PROFILE: development
    JAVA_OPTS: "-Djava.security.egd=file:/dev/./urandom -XX:+UseSerialGC -Dspring.profiles.active=${PROFILE} -Dserver.port=${PORT}" # Add "-XX:+UseSerialGC" in JAVA_OPTS to enable GC
    BP_JVM_VERSION: 11 #To explicitly set Java Version
```

Deployment uses paketo buildpacks which will detect the builder, build the application and deploy.

For more buildpack specific cofiguration refer documentation for [paketo java buildpack](https://paketo.io/docs/buildpacks/language-family-buildpacks/java/#)

## .cfignore

Buildpack will handle the responsibility of build and deploying the application, so the directories containing build artificats don't need to be pushed, and the configuration will looks like this.

```txt
build
target
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
   *** Images (sha256:da5d0ecf95c0fbb001af5fa3010424da14c7953e1ce46cb1a530eefa02abd9ab):
   core.registry.openxcell.dev/cf-for-k8s/41abb284-55e6-46d0-a53a-a64420b28f94
   core.registry.openxcell.dev/cf-for-k8s/41abb284-55e6-46d0-a53a-a64420b28f94:b11.20210601.184539
   Reusing cache layer 'paketo-buildpacks/bellsoft-liberica:jdk'
   Adding cache layer 'paketo-buildpacks/gradle:application'
   Adding cache layer 'paketo-buildpacks/gradle:cache'
   Build successful
   ...
   ...

```

> Output will print url for your Application, for our app `my-java-app` the url will be `my-java-app.apps.openxcell.dev`


## Actuator Healthcheck

Cloudfoundry follows 12-Factor Application Principles and it is preferred to configure health checks into your application.

Spring Boot Actuator is a library that provides easy to configure health check endpoints. 

Configuring the Dependency:

<!-- tabs:start -->

## **Maven**

Add this to dependencies in `pom.xml` file

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## **Gradle**

Add this to dependencies in `build.gradle` file

```java
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
<!-- tabs:end -->

Actuator by default enables health check endpoint without any configuration at `/actuator/health`, which would return status of the application

```bash
curl localhost:8080/actuator/health
{"status":"UP"}
```

For more configuration details check out this [Tutorial](https://www.baeldung.com/spring-boot-actuators)

You may also check this [demo application](https://demo-java.apps.openxcell.dev/actuator/) . Its source code is available [here](https://github.com/callicoder/spring-boot-actuator-demo)