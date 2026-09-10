pipeline {
  agent {
    node {
      label 'kaniko-agent'
    }
  }

  stages {
    stage('Smoke Test') {
      steps {
        container('kaniko') {
          sh '''
            mkdir -p /workspace
            cat <<'EOF' > /workspace/Dockerfile
FROM alpine
CMD ["echo", "hello"]
EOF
            /kaniko/executor --context=/workspace \
              --dockerfile=/workspace/Dockerfile \
              --destination=jamencidor/flaskapp:smoke-test
          '''
        }
      }
    }
  }
}
