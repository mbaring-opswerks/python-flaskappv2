pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: kaniko
                image: gcr.io/kaniko-project/executor:debug
                command: ["sleep"]
                args: ["99d"]
                volumeMounts:
                - name: regcred
                  mountPath: /kaniko/.docker
              volumes:
              - name: regcred
                secret:
                  secretName: regcred
                  items:
                  - key: .configjson
                    path: config.json
            '''
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

