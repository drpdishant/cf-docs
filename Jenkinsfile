def cf_api = "https://api.cloud.openxcell.dev/v3"
def cf_org = "openxcell"
def cf_space = "devops"

pipeline {
    agent {
         docker {
            image "ppiper/cf-cli"
            label 'ec2-fleet'
        }
    } 
    stages {
        stage('Push to Cloudfoundry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'cloudfoundry_devops', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh "cf login -a ${cf_api} -u ${USERNAME} -p ${PASSWORD}"
                sh "cf push"
                }
            }
        }
    }
}
