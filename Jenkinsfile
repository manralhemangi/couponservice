pipeline {
    agent any
    
    tools {
        maven 'maven'
    }
    
    environment {
        // Environment variables for credentials and Git repository
        GIT_REPO = 'https://github.com/manralhemangi/couponservice'
        GIT_CREDENTIALS_ID = 'manralhemangi'
        DOCKER_HUB_USER = 'manralhemangi'  // Update this
        APP_IMAGE = 'couponservice'
        DB_IMAGE = 'mysql:8'  // Keeping the official MySQL image

    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from Git repository
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],  // Checkout the 'main' branch
                    userRemoteConfigs: [
                        [
                            url: "${GIT_REPO}",  // Git repository URL
                            credentialsId: "${GIT_CREDENTIALS_ID}"  // Jenkins credentials ID
                        ]
                    ]
                ])
            }
        }
        
        stage('Build') {
            steps {
                echo 'Running Maven clean install...'
                bat 'mvn clean install' // Runs the Maven clean install 
            }
        }
        
          stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        
        // stage('SonarQube Analysis') {
        //     environment {
        //         SONAR_HOST_URL = "http://localhost:9000"
        //         SONAR_AUTH_TOKEN = credentials('SonarQubeNew')
        //     }
        //  steps{
        //          bat "mvn sonar:sonar -Dsonar.projectKey=InterestCalculatorPipeline -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN"
        //     }
        // }
        
           stage('Build and Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker_credentials', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {                   
                    // bat "docker login -u hemangimanral -p ${DOCKER_HUB_PASS}"
                    bat "echo ${DOCKER_HUB_PASS} | docker login -u ${DOCKER_HUB_USER} --password-stdin"

                    
                    echo 'Building Docker image for the application...'
                    bat "docker build --no-cache -t ${DOCKER_HUB_USER}/${APP_IMAGE}:latest ."

                    echo 'Pushing application image to Docker Hub...'
                    bat "docker push ${DOCKER_HUB_USER}/${APP_IMAGE}:latest"
                    
                }
            }
        }

        // stage('Deploy with Docker Compose') {
        //     steps {
        //         script {
        //             echo 'Stopping existing containers...'
        //             bat 'docker-compose down'

        //             echo 'Pulling latest images...'
        //             bat "docker pull ${DOCKER_HUB_USER}/${APP_IMAGE}:latest"

        //             echo 'Starting new deployment...'
        //             bat 'docker-compose up -d'
                    
        //             echo 'Showing docker compose logs'
        //             bat 'docker-compose logs'
        //         }
        //     }
        }
         
    }
}