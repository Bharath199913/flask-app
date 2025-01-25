pipeline { 
    agent any 

    environment { 
        SONARQUBE_SERVER = 'SQ'  // Name of your SonarQube server in Jenkins 
        DOCKER_IMAGE = 'helloworld-app:latest' // Replace with your desired image name
        IMAGE_TAG = 'latest' 
        IMAGE_NAME = 'helloworld-app' // Image name
        GCR_PROJECT_ID = 'banded-setting-448905-d6' // Replace with your GCP project ID
        GCR_REGISTRY = "gcr.io"
        GCR_REPO = "${GCR_REGISTRY}/${GCR_PROJECT_ID}/${IMAGE_NAME}"
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

        stage('Authenticate with GCR') {
            steps {
                // Use the updated Google Service Account key stored as FileCredentials
                withCredentials([file(credentialsId: 'gcr-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    // The plugin will automatically set GOOGLE_APPLICATION_CREDENTIALS to the JSON key file
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh 'gcloud config set project $GCR_PROJECT_ID'
                }
            }
        }
         stage('Push Docker Image to GCR') {
            steps {
                script {
                    // Tag the Docker image
                    sh "docker tag $DOCKER_IMAGE $GCR_REPO:$IMAGE_TAG"
                    // Push the image to GCR
                    sh "docker push $GCR_REPO:$IMAGE_TAG"
                }
            }
        }
    }
}
