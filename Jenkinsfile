pipeline {
    agent any
    
    environment {
        GITHUB_REPO_URL = 'https://github.com/halephu01/Jenkins-CI-CD.git'
        BRANCH_NAME = 'main'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}",
                    url: "${GITHUB_REPO_URL}"
            }
        }
        
        stage('Build and Deploy') {
            steps {
                sh 'docker-compose up'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}


