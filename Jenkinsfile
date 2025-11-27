pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
    }
    
    options {
        // Timeout counter starts after agent is allocated
        timeout(time: 1, unit: 'HOURS')
    }
    
    environment {
        APP_ENV = "DEV"
    }
    
    stages {
        // ❌ SUPPRIMÉ: stage('Code Checkout') - Jenkins le fait déjà automatiquement
        
        stage('Code Build') {
            steps {
                sh 'mvn install -Dmaven.test.skip=true'
            }
        }
    }
    
    post {
        always {
            echo "======always======"
        }
        success {
            echo "=====pipeline executed successfully ====="
        }
        failure {
            echo "======pipeline execution failed======"
        }
    }
}
