pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'aziz/student-management-app'
        SONAR_HOST_URL = 'http://localhost:9000'
        // Le token SonarQube doit être configuré dans Jenkins Credentials
        SONAR_TOKEN = credentials('sonarqube-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo "Building with Maven..."
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                script {
                    // Utilisation du SonarQube Scanner
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.projectName='Student Management Application' \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN} \
                            -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo "Checking SonarQube Quality Gate..."
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Package') {
            steps {
                echo "Packaging application..."
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker build -t ${DOCKER_IMAGE}:latest .
                    docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:${BUILD_NUMBER}
                """
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying container..."
                sh """
                    docker stop student-app || true
                    docker rm student-app || true
                    docker run -d --name student-app -p 8081:8089 \
                        -e SPRING_DATASOURCE_URL=jdbc:mysql://192.168.33.1:3306/studentdb \
                        -e SPRING_DATASOURCE_USERNAME=jenkins \
                        -e SPRING_DATASOURCE_PASSWORD=jenkins123 \
                        ${DOCKER_IMAGE}:latest
                """
            }
        }
    }
    
    post {
        always {
            echo "======Pipeline completed======"
            // Nettoyer l'espace de travail
            cleanWs()
        }
        success {
            echo "=====Pipeline executed successfully ====="
            echo "Application deployed at: http://192.168.33.10:8081/student"
            echo "SonarQube report: ${SONAR_HOST_URL}/dashboard?id=student-management"
        }
        failure {
            echo "======Pipeline execution failed======"
        }
    }
}
