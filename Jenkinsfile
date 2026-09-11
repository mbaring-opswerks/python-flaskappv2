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

  - name: git
    image: alpine/git
    command:
    - sleep
    args:
    - 99d

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

        DEPLOYMENT_REPO = 'https://github.com/mbaring-opswerks/deployment-configv2.git'

        DEPLOYMENT_FILE = 'apps/flask-app/deployment.yaml'
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

        stage('Update deployment-config') {
            steps {
                container('git') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: '0d98625f-c43a-48de-a423-263930723728',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                            set -e
                            set +x

                            cat > /tmp/git-askpass.sh <<'EOF'
#!/bin/sh
case "$1" in
    *Username*) echo "$GIT_USERNAME" ;;
    *Password*) echo "$GIT_PASSWORD" ;;
esac
EOF

                            chmod 700 /tmp/git-askpass.sh

                            export GIT_ASKPASS=/tmp/git-askpass.sh
                            export GIT_TERMINAL_PROMPT=0

                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@://github.com deployment-configv2

                            cd deployment-configv2

                            # Targets '- image: raycojp/flask-app:<TAG>' while preserving indentations
                            sed -i "s|- image: ${DOCKER_IMAGE}:.*|- image: ${DOCKER_IMAGE}:${IMAGE_TAG}|" "${DEPLOYMENT_FILE}"

                            git config user.name "Jenkins"
                            git config user.email "jenkins@localhost"

                            echo "Updated deployment configuration:"
                            git diff -- "${DEPLOYMENT_FILE}"

                            # Safety check: fail if sed failed to update the file
                            if [ -z "$(git status --porcelain)" ]; then
                                echo "ERROR: No changes detected in ${DEPLOYMENT_FILE}. Check image string match."
                                exit 1
                            fi

                            git add "${DEPLOYMENT_FILE}"

                            git commit \
                                -m "Update flask-app image to ${IMAGE_TAG}"

                            git push origin main

                            rm -f /tmp/git-askpass.sh
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "CI completed successfully."
            echo "Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
            echo "deployment-config updated successfully."
            echo "CI -> CD handoff complete. Flux can now reconcile the change."
        }

        failure {
            echo "Pipeline failed."
            echo "No later stages were executed after the failure."
        }
    }
}
