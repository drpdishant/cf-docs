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

`
include:
  remote: 'https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/raw/master/templates/build.yaml'
`

It is basically add hidden jobs in your CI/CD . You can call any job by using **extends** keyword

### For API

Use [Gitlab CI Templates](https://gitlab.orderhive.plus/public-resources/gitlab-ci/-/blob/master/templates) for your reference

Now, We will learn each stage one by one :

#### get_env

Use this stage/job if you .env file is coming from s3 bucket

For this stage, first you have to setup ci/cd variables for Access Key and Secret Key which is shown above

**Example :**

`
get_env:
  stage: get_env
  extends: .get_env
  variables:
    BUCKET_NAME: "__development_bucket__"
    PRODUCTION_BUCKET_NAME: "__production_bucket__"
`
As you can see , you have to add your .env full path ex. **BUCKET_NAME: "openxcell-development-private/__project__/.env"** . Now,you will use this .env for next stage. Enjoy ....








