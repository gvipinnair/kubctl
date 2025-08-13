pipeline {
    agent any

    environment {
        IMAGE_NAME = "vipingnair/nginx"
        NAMESPACE = "vipin"
        DEPLOYMENT_FILE = "k8s/deployment.yaml"
        SERVICE_FILE = "k8s/service.yaml"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/gvipinnair/kubctl.git', branch: 'main'
                script {
                    def commitMessage = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                    def versionTag = commitMessage.tokenize(':')[-1].trim()
                    if (!versionTag) error "Commit message doesn't contain a version tag"
                    env.IMAGE_TAG = versionTag
                    echo "🛠 Using image tag: ${IMAGE_TAG}"
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "sudo nerdctl build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        // Log in to Docker Hub
                        sh "echo \$DOCKER_PASS | nerdctl login -u \$DOCKER_USER --password-stdin"

                        // Tag the image
                        sh "nerdctl tag ${IMAGE_NAME}:${IMAGE_TAG} \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}"

                        // Push the image
                        sh "nerdctl push \$DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Replace placeholders in deployment.yaml
                    sh """
                        sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' ${DEPLOYMENT_FILE}
                        sed -i 's|namespace: .*|namespace: ${NAMESPACE}|g' ${DEPLOYMENT_FILE}
                    """

                    // Apply the deployment and service
                    sh """
                        kubectl apply -f ${DEPLOYMENT_FILE}
                        kubectl apply -f ${SERVICE_FILE}
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify the deployment and service
                    sh """
                        kubectl get deployments -n ${NAMESPACE}
                        kubectl get services -n ${NAMESPACE}
                        kubectl get pods -n ${NAMESPACE}
                    """
                }
            }
        }
    }
}
