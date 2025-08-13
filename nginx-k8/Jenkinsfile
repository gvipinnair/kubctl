pipeline {
    agent any

    stages {
        stage('Deploy NGINX to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}