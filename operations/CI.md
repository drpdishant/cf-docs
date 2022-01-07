## Gitlab CI/CD

Go to **Settings > Expand Visibility, project features, permissions > Enable CI/CD**

![Enable CI](./enable-ci.PNG ':size=70%')


**Note-** CI/CD Runner only work with **Internal/Public** projects

### Make Project Internal/Public

Go to **Setting > Expand Visibility, project features, permissions > Make Project Internal**

![Project Visibility](./project-visible.png ':size=70%')


Make Sure that if there is any parent repository or main repository available also make that repository Internal/Public


**Now you can see CI/CD Tab on side-navigation bar**

### How to add CI/CD variables in your pipeline

Go to **Setting > CI/CD Settings > Expand Variables > Add Variable**

![Add CI/CD Variable](./add-variable.PNG ':size=70%')

**Note** - For add SSH KEY . just change variable Type to File

## Make .gitlab-ci.yml file in your repository

### import templates

This is required for all pipeline

```
include:
  remote: 'https://${CI_SERVER_HOST}/public-resources/gitlab-ci/-/raw/master/templates/build.yaml'

```

It is basically add hidden jobs in your CI/CD . You can call any job by using **extends** keyword

### For API

Use [Gitlab CI API Template](https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/raw/master/templates/.gitlab-ci-api.yml) for your reference

First add following into CI/CD file :
```
stages:
  - get_env
  - build
  - deploy
variables:
  PROJECT: "__PROJECT_NAME__"
  TECHNOLOGY: "__TECHNOLOGY__" # java or nodejs
```

Now, We will learn each stage one by one :

#### get_env(optional stage)

Use this stage/job if you .env file is coming from s3 bucket

For this stage, first you have to setup ci/cd variables for Access Key and Secret Key which is shown above

**Variables to be store in CI/CD variables :**

ACCESS_KEY_$CI_COMMIT_REF_NAME
SECRET_KEY_$CI_COMMIT_REF_NAME

Here CI_COMMIT_REF_NAME is branch name like if you want to add ACCESS_KEY for development brannch use **ACCESS_KEY_development="Afnmdfn"** in your CI/CD variables


**Example :**

```
get_env:
  stage: get_env
  extends: .get_env
  variables:
    BUCKET_NAME: "__development_bucket__"
    PRODUCTION_BUCKET_NAME: "__production_bucket__"
```
As you can see , you have to add your .env full path ex. **BUCKET_NAME: "openxcell-development-private/__project__/.env"** . Now,you will use this .env for next stage. Enjoy ....



#### build

Use this stage/job for api services like java,nodejs. 

**Variables used**

**BUILD_ARGS** - give build arguments using this varible.

For node js use this :
```
BUILD_ARGS: "--build-arg APP_NAME=${PROJECT} --build-arg NODE_ENV=${CI_COMMIT_REF_NAME}"
```

For java use this:
```
BUILD_ARGS: "--build-arg APP_NAME=${PROJECT} --build-arg PROFILE=${CI_COMMIT_REF_NAME}"
```

**DOCKERFILE_PATH** - Use for declare dockerfile full path (Default value is **./Dockerfile**)

**ENV_FILE** - .env full path (Default value is **./.env.$CI_COMMIT_REF_NAME**)

**IMAGE_TAG** - image tag (Default value is **$CI_COMMIT_REF_NAME**)

Add following for build stage:

```
build:
  stage: build
  extends: .build
  variables:
    BUILD_ARGS: "--build-arg APP_NAME=${PROJECT} --build-arg NODE_ENV=${CI_COMMIT_REF_NAME}"
```

build stage is mainly common for both development, master branch. We will learn , how to run specific job for specific branch in deploy stage.


#### deploy

In this stage we will deploy our api/backend to our api server.

**Note** - For production , store SSH key into group variable or project variable **as file**

![Add SSH Key](./add-ssh-key.png ':size=70%')

**Variables used**

**CONT_PORT(Required)** - port of your api.

**DEPLOY_PATH** - where you want to store your docker-compose file . Default value is **/srv/www/$TECHNOLOGY/$PROJECT**

**IP_ADDRESS** - ip of server . Default value is **api.openxcell.dev**

**DOCKER_COMPOSE_TEMPLATE** - template which you want to use <br>
[Development docker-compose](https://gitlab.orderhive.plus/-/snippets/15/raw) // use this for development server <br>
[Production/Master docker-compose](https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/raw/master/templates/docker/docker-compose.yaml)  //for production server

**SSH_KEY_NAME** - ssh key name which you have stored in CI/CD variable. Default value is  **SSH_PRIVATE_KEY_DEV** which is for development server

**IMAGE_TAG** - image tag which you have given in build stage. Default value is **$CI_COMMIT_REF_NAME** (Branch Name)

**Example:**
```
deploy_dev:
  stage: deploy
  extends: .deploy
  variables:
    CONT_PORT: "7272"
  only:
    - development

deploy_prod:
  stage: deploy
  extends: .deploy
  environment:
    name: $CI_COMMIT_REF_NAME
    url: "__url__" #URL of production server
  variables:
    CONT_PORT: "7272"
    DEPLOY_PATH: $PROJECT
    IP_ADDRESS: "__prod_ip__"
    SSH_KEY_NAME: "__prod_key_name__" # Add key name not its value
  only:
    - master
```
Here you can see **environment**. It is basically used for **Deployments > Environments** in gitlab. In this you have to give name and url of your production environment . For development environment , it is already set as default. 

Here we are using **only** keyword to specify branch . Like if you want to run specific job for specific branch only you can use this like above.


### For Front-End(ReactJS)

Use [Gitlab CI API Template](https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/raw/master/templates/.gitlab-ci-static.yml) for your reference

First add following into CI/CD file :

```
include:
  remote: 'https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/raw/master/templates/build.yaml'
stages:
  - build
  - deploy
variables:
  NGINX_PATH: "/home/ubuntu/docker-stack/nginx_conf/react" #required in development branch 
  PROJECT: "__project-name__"
```
Now, We will learn each stage one by one :

#### build 

This stage will build your reactjs project and store it into artifacts. For more about artifacts follow this link [Gitlab Artifacts](https://docs.gitlab.com/ee/ci/pipelines/job_artifacts.html)

**Variables used**

**ENV_FILE** - .env full path (Default value is **./.env.$CI_COMMIT_REF_NAME**)

**Example:**
```
build: 
  stage: build
  extends: .build_static
  variables:
    ENV_FILE: .env-live 
```


#### deploy

This stage will deploy your static code to development/production server

**Variables used**

**DEPLOY_PATH** - where you want to store your docker-compose file . Default value is **/srv/www/reactjs/$PROJECT**

**IP_ADDRESS** - ip of server . Default value is **api.openxcell.dev**

**SSH_KEY_NAME** - ssh key name which you have stored in CI/CD variable. Default value is  **SSH_PRIVATE_KEY_DEV** which is for development server

**Example:**
```
deploy_prod:
  stage: deploy
  environment:
    name: production
    url: "__url__"
  variables:
    DEPLOY_PATH: /usr/share/nginx/html
    IP_ADDRESS: "3.216.179.32"
    SSH_KEY_NAME: "KEY_PROD"
  only:
    - production
```

## Now you are ready to rock!!!!!

You can see running pipeline and its logs in **CI/CD > Pipelines or Jobs**

![Pipeline](./pipeline.png ':size=70%')



