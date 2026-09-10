pipeline {
    agent {
        node {
            label 'kaniko-agent'
        }
    }
    stages {
        stage('smoke test') {
            steps {
                container('kaniko') {
                    sh '''
                        mkdir -p /workspace

                        cat <<'EOF' > /workspace/Dockerfile
FROM alpine
CMD ["echo", "hello"]
EOF

                        /kaniko/executor \
                          --context=/workspace \
                          --dockerfile=/workspace/Dockerfile \
                          --destination=jrayco/flask-app:v1-mweh
                    '''
                }
            }
        }
    }
}


