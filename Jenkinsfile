def cf_api = "https://api.cloud.openxcell.dev/v3"
def cf_org = "openxcell"
def cf_space = "devops"

pipeline {
    agent {
        label 'ec2-fleet'
    } 
    stages {
        stage('Push to Cloudfoundry') {
            steps {
            pushToCloudFoundry(
                target: cf_api,
                organization: cf_org,
                cloudSpace: cf_space,
                credentialsId: 'cloudfoundry_devops'
                )
            }
        }
    }
}
