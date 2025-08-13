pipeline {
    agent any

    environment {
        IMAGE_NAME = "vipingnair/nginx"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/gvipinnair/kubctl.git', branch: 'main'

                script {
                    // Extract last commit message
                    def commitMessage = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()

                    // Extract tag from message (assumes format like: deploy: version-tag)
                    def versionTag = commitMessage.tokenize(':')[-1].trim()
                    
                    if (!versionTag) {
                        error "Commit message doesn't contain a version tag (e.g., 'deploy: test1')"
                    }

                    env.IMAGE_TAG = versionTag
                    echo "🛠 Using image tag: $IMAGE_TAG"
                }
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                   sudo nerdctl build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | sudo  nerdctl login -u $DOCKER_USER --password-stdin
                       sudo  nerdctl push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "📦 Deploying to K8s with image $IMAGE_NAME:$IMAGE_TAG"
                    echo "hi"
                    sed -i "s|image: .*|image: $IMAGE_NAME:$IMAGE_TAG|g" k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                '''
            }
        }
    }
}
