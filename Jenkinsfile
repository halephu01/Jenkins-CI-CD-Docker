pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        
        USER_SERVICE_IMAGE = 'halephu01/user-service'
        FRIEND_SERVICE_IMAGE = 'halephu01/friend-service'
        AGGREGATE_SERVICE_IMAGE = 'halephu01/aggregate-service'        
        SONAR_TOKEN = credentials('sonar')
        SONAR_PROJECT_KEY = 'microservices-project'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/halephu01/Jenkins-CI-CD.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build Services') {
            parallel {
                stage('Build User Service') {
                    steps {
                        dir('user-service') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
                
                stage('Build Friend Service') {
                    steps {
                        dir('friend-service') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
                
                stage('Build Aggregate Service') {
                    steps {
                        dir('aggregate-service') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    def services = ['user-service', 'friend-service', 'aggregate-service']

                    withSonarQubeEnv('SonarScanner') {
                        services.each { service ->
                            dir(service) {
                                sh """
                                    ${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey=${service} \
                                    -Dsonar.projectName=${service} \
                                    -Dsonar.sources=. \
                                    -Dsonar.java.binaries=target/classes \
                                """
                            }
                        }
                    }
                }
            }
        }
        
        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }        
        
        stage('Build and Push Docker Images') {
            steps {
                script {
                    def services = ['user-service', 'friend-service', 'aggregate-service']
                    
                    services.each { service ->
                        echo "Building and pushing ${service} Docker image..."
                        try {
                            sh """
                                docker build -t ${service} -f ${service}/Dockerfile .
                                docker tag ${service} halephu01/${service}:latest
                            """
                            
                            sh """
                                echo "Pushing ${service} image..."
                                docker push halephu01/${service}:latest
                            """
                            
                            echo "Successfully built and pushed ${service} image"
                        } catch (Exception e) {
                            echo "Error processing ${service}: ${e.message}"
                            throw e
                        }
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    sh """
                        sed -i 's|image: halephu01/user-service:.*|image: ${USER_SERVICE_IMAGE}:${VERSION}|' docker-compose.yml
                        sed -i 's|image: halephu01/friend-service:.*|image: ${FRIEND_SERVICE_IMAGE}:${VERSION}|' docker-compose.yml
                        sed -i 's|image: halephu01/aggregate-service:.*|image: ${AGGREGATE_SERVICE_IMAGE}:${VERSION}|' docker-compose.yml
                    """

                    sh 'docker-compose down || true'
                    sh 'docker-compose up -d'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
        }
        always {
            script {
                try {
                    sh """
                        docker-compose down || true
                        docker logout
                        
                        # Remove images
                        for service in ${env.DOCKER_IMAGES}; do
                            image="halephu01/\${service}:${BUILD_NUMBER}"
                            if docker image inspect \${image} >/dev/null 2>&1; then
                                echo "Removing image \${image}..."
                                docker rmi \${image} || true
                            fi
                        done
                    """
                } catch (Exception e) {
                    echo "Cleanup failed: ${e.message}"
                }
            }
        }
    }
} 