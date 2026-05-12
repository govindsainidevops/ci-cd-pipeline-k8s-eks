pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "govindsaini442/devops-app"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        // Disable temporarily
        stage('Deploy to Kubernetes') {
            when {
                expression { false }
            }
            steps {
                sh 'kubectl apply -f k8s-deployment.yaml'
            }
        }
    }
}