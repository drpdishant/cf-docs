def cf_api = "api.cloud.openxcell.dev"
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
                    sh "cf target -o ${cf_org} -s ${cf_space}"
                    sh "cf push"
                }
            }
        }
    }
}
