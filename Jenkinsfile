pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        IMAGE_NAME = "shek07/ecommarce"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "e-commarce-app"

        DOCKER_USER = credentials('dockerhub-credentials')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging application...'
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'

                sh """
                docker build \
                -t ${IMAGE_NAME}:${IMAGE_TAG} \
                -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Docker Login') {
            steps {
                sh '''
                echo "$DOCKER_USER_PSW" | docker login -u "$DOCKER_USER_USR" --password-stdin
                '''
            }
        }

        stage('Docker Push') {
            steps {
                sh """
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                docker push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application with Docker Compose V2...'
                
                    sh 'docker-compose down'
                    sh 'docker-compose up -d --build'
		
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}
