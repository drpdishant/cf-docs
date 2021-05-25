def cf_api = "api.cloud.openxcell.dev"
def cf_org = "openxcell.org"
def cf_space = "devops"

pipeline {
    pipeline {
    agent {
        label 'ec2-fleet'
    } 
    stages {
        stage('Push to Cloudfoundry') {
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