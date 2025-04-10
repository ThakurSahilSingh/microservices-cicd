pipeline {
    agent any
    environment {
        REGISTRY = "sahil0824"
        IMAGE_NAME = "java-microservice"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean install'
            }
        }

        // Conditional Docker Build + Push only for develop
        stage('Docker Build & Push') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    sh 'docker build -t $REGISTRY/$IMAGE_NAME:$BRANCH_NAME .'
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $REGISTRY/$IMAGE_NAME:$BRANCH_NAME
                        '''
                    }
                }
            }
        }

        stage('K8s Deploy') {
            when {
                branch 'develop'
            }
            steps {
                withKubeConfig([credentialsId: 'k8s-creds']) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }
}
