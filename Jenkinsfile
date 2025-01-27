pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SQ'  // Name of your SonarQube server in Jenkins
        DOCKER_IMAGE = 'helloworld-app:latest' // Replace with your desired image name
        IMAGE_TAG = 'latest'
        IMAGE_NAME = 'helloworld-app' // Image name
        DOCKERHUB_USERNAME = 'Bharath1304'  // Replace with your DockerHub username
        DOCKERHUB_REPO = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}" // DockerHub repository
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // Jenkins DockerHub credentials ID
    }

    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                sh 'trivy image $DOCKER_IMAGE'
            }
        }

        stage('Authenticate with DockerHub') {
            steps {
                // Authenticate with DockerHub
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    // Tag the Docker image for DockerHub
                    sh "docker tag $DOCKER_IMAGE $DOCKERHUB_REPO:$IMAGE_TAG"
                    
                    // Push the image to DockerHub
                    sh "docker push $DOCKERHUB_REPO:$IMAGE_TAG"
                    
                    // Confirm the image is in DockerHub
                    sh 'docker images'
                }
            }
        }
    }
}
