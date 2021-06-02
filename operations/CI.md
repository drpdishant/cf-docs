# Configuring CI

In a CI Pipeline we can automate deployment to cloudfoundry using `ppiper/cf-cli` docker image

## Jenkins

A sample ci stage in jenkins declarative pipeline would look as shown

```groovy
stage('Deploy CF') {
            when {
                branch development_branch
            }
            agent {
                docker {
                image "ppiper/cf-cli"
                label 'ec2-fleet'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'cloudfoundry_devops', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh "cf login -a $cf_api -u $USERNAME -p $PASSWORD"
                    sh "cf target -o $cf_org -s $cf_space"
                    sh "cf push"
                }
                cleanWs()
            } 
        }
```

- You will need to define variables globally at beginning of the Jenkinsfile

```groovy
def cf_api = "api.cloud.openxcell.dev" #API endpoint
def cf_org = "openxcell" # Org Name
def cf_space = "development" # Space
```