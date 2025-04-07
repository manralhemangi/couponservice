pipeline {
    agent any
    
    tools {
        maven 'maven'
    }
    
    environment {
        GIT_REPO = 'https://github.com/manralhemangi/couponservice'
        GIT_CREDENTIALS_ID = 'manralhemangi'
        DOCKER_HUB_USER = 'hemangimanral'
        APP_IMAGE = 'couponservice'
        DB_IMAGE = 'coupondb'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: "${GIT_REPO}",
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }
        
        stage('Build') {
            steps {
                echo 'Running Maven clean install...'
                bat 'mvn clean install'
            }
        }
        //test
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        // Uncomment if using SonarQube
        // stage('SonarQube Analysis') {
        //     environment {
        //         SONAR_HOST_URL = "http://localhost:9000"
        //         SONAR_AUTH_TOKEN = credentials('SonarQubeNew')
        //     }
        //     steps {
        //         bat "mvn sonar:sonar -Dsonar.projectKey=couponService -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN"
        //     }
        // }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker_credentials', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {                   
                        bat "docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASS}"
                        
                        echo 'Building Docker image for the application...'
                        bat "docker build --no-cache -t ${DOCKER_HUB_USER}/${APP_IMAGE}:latest ."

                        echo 'Pushing application image to Docker Hub...'
                        bat "docker push ${DOCKER_HUB_USER}/${APP_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {
                    echo 'Stopping existing containers...'
                    bat 'docker compose down -v'

                    echo 'Pulling latest images...'
                    bat "docker pull ${DOCKER_HUB_USER}/${APP_IMAGE}:latest"


                }
            }
        }
    }
}
