pipeline { 
    agent any 

    environment { 
        SONARQUBE_SERVER = 'SQ'
        DOCKER_IMAGE = 'helloworld-app:latest' 
        IMAGE_TAG = 'latest'
        IMAGE_NAME = 'helloworld-app' 
        GCR_PROJECT_ID = 'banded-setting-448905-d6'
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
                sh "docker build -t ${DOCKER_IMAGE} ." 
            } 
        }

        stage('Scan Docker Image with Trivy') { 
            steps { 
                sh "trivy image ${DOCKER_IMAGE}" 
            } 
        }

        stage('Authenticate with GCR') {
            steps {
                withCredentials([file(credentialsId: 'gcr-service-account-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud config set project ${GCR_PROJECT_ID}
                    """
                }
            }
        }

        stage('Push Docker Image to GCR') {
            steps {
                script {
                    // Tag Docker image
                    sh "docker tag ${DOCKER_IMAGE} ${GCR_REPO}:${IMAGE_TAG}"
                    
                    // Assign IAM roles (only once; remove duplicates)
                    sh """
                        gcloud projects add-iam-policy-binding ${GCR_PROJECT_ID} \
                        --member="serviceAccount:spinnaker-front50-sa@${GCR_PROJECT_ID}.iam.gserviceaccount.com" \
                        --role="roles/serviceusage.serviceUsageAdmin"

                        gcloud projects add-iam-policy-binding ${GCR_PROJECT_ID} \
                        --member="serviceAccount:spinnaker-front50-sa@${GCR_PROJECT_ID}.iam.gserviceaccount.com" \
                        --role="roles/storage.objectAdmin"
                    """

                    // Push the Docker image
                    sh "docker push ${GCR_REPO}:${IMAGE_TAG}"

                    // Verify artifact repositories
                    sh "gcloud artifacts repositories list"
                }
            }
        }
    }
}
