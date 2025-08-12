pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // Jenkins credential ID for Docker Hub username/password
        DOCKER_IMAGE = "vipingnair/java-app"       // Your Docker image name
        GIT_REPO = 'https://github.com/gvipinnair/kubctl.git'
        K8S_NAMESPACE = 'default'                   // Change if using a different namespace
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  kubectl set image deployment/demo-deployment demo-container=${DOCKER_IMAGE}:latest -n ${K8S_NAMESPACE}
                  kubectl rollout status deployment/demo-deployment -n ${K8S_NAMESPACE}
                '''
            }
        }
    }
}