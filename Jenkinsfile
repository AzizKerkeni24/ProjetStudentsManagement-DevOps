pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
    }
    
    environment {
        APP_ENV = "DEV"
        DOCKER_IMAGE = "student-management-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Code Build') {
            steps {
                echo "Building with Maven..."
                sh 'mvn clean install -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }
        
        stage('List Docker Images') {
            steps {
                echo "Listing Docker images..."
                sh 'docker images | grep student-management-app || true'
            }
        }
        
        stage('Deploy Container') {
            steps {
                echo "Deploying container..."
                sh """
                    docker stop student-app || true
                    docker rm student-app || true
                    docker run -d --name student-app -p 8081:8080 ${DOCKER_IMAGE}:latest
                """
            }
        }
    }
    
    post {
        always {
            echo "======Pipeline completed======"
        }
        success {
            echo "=====Pipeline executed successfully ====="
            echo "Application deployed at: http://192.168.33.10:8081"
        }
        failure {
            echo "======Pipeline execution failed======"
        }
    }
}
