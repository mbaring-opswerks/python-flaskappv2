pipeline {
    agent {
        node {
            label 'kaniko-agent'
        }
    }
    stages {
        stage('Build and Push') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                          --context="${WORKSPACE}" \
                          --dockerfile="${WORKSPACE}/Dockerfile" \
                          --destination=raycojp/flask-app:v1-hehehe
                    '''
                }
            }
        }
    }
}

