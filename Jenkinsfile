pipeline {
    agent any

    stages {
        stage('Deploy NGINX to Kubernetes') {
            steps {
                dir('nginx-k8') {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}