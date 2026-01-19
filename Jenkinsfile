pipeline {
    agent any

    environment {
        // Docker Hub credentials ID stored in Jenkins Credentials
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' 
        DOCKER_HUB_USERNAME = 'kiranreddy120895'
        IMAGE_NAME = 'java-shopping-app'
        K8S_NAMESPACE = 'default' // Change if needed
        K8S_DEPLOYMENT_FILE = 'k8s-deployment.yaml'
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/madhu-kiran12/java-shopping-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:latest")
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update image in k8s deployment YAML before apply
                    sh """
                    kubectl set image -f ${K8S_DEPLOYMENT_FILE} java-shopping-app=${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:latest --namespace=${K8S_NAMESPACE} || kubectl apply -f ${K8S_DEPLOYMENT_FILE} --namespace=${K8S_NAMESPACE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}

