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
  remote: 'https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/raw/master/templates/build.yaml'

```

It is basically add hidden jobs in your CI/CD . You can call any job by using **extends** keyword

### For API

Use [Gitlab CI Templates](https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/blob/master/templates) for your reference

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







