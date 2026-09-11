pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  namespace: devops-tools
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/sleep
    args:
    - 99d
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker

  volumes:
  - name: docker-config
    secret:
      secretName: regcred
      items:
      - key: .dockerconfigjson
        path: config.json
'''
        }
    }

    environment {
        DOCKER_IMAGE = 'raycojp/flask-app'
    }

    stages {

        stage('Get Commit Hash') {
            steps {
                script {
                    env.COMMIT_SHA = sh(
                        script: 'git rev-parse HEAD',
                        returnStdout: true
                    ).trim()

                    env.COMMIT_5 = env.COMMIT_SHA.substring(
                        env.COMMIT_SHA.length() - 5
                    )

                    env.IMAGE_TAG = "${BUILD_NUMBER}-${env.COMMIT_5}"

                    echo "Commit SHA: ${env.COMMIT_SHA}"
                    echo "Image tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build and Push Image') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                          --context="${WORKSPACE}" \
                          --dockerfile="${WORKSPACE}/Dockerfile" \
                          --destination="${DOCKER_IMAGE}:${IMAGE_TAG}"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Image pushed: ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
