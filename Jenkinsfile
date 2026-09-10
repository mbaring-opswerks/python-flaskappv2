pipeline {
    agent {
        node {
            label 'kaniko'
        }
    }
    stages {
        stage('Build and Push') {
            steps {
                sh '''
                    /kaniko/executor \
                      --context=dir://. \
                      --dockerfile=Dockerfile \
                      --destination=raycoJp/flask-app:v1-hehehe
                '''
            }
        }
    }
}

