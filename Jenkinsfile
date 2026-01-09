pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "pratikphanase97/multibranch-flask-app"
        GIT_USER = "pratikphanase"
        GIT_EMAIL = "pratikphanase@gmail.com"
    }

    stages {

        stage("Checkout Code") {
            steps {
                checkout scm
            }
        }

        stage("Build and Push image") {
            when { branch "main" }
            steps {
                script {
                    def IMAGE_TAG = "build-${BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                    env.IMAGE_TAG = IMAGE_TAG
                }
            }
        }

        stage("Update K8s Manifests") {
            when { branch "main" }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-cred',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        set -e
                        git config user.name "${GIT_USER}"
                        git config user.email "${GIT_EMAIL}"
                        git fetch origin main
                        git checkout main
                        git reset --hard origin/main
                        sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                        git add k8s/deployment.yaml
                        git diff --cached --quiet || git commit -m "Update k8s deployment image to ${IMAGE_NAME}:${IMAGE_TAG}"
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/pratikphanase/Production-Grade-Deployment.git main
                        """
                    }
                }
            }
        }
    }
}

