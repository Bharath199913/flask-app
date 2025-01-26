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
        GOOGLE_APPLICATION_CREDENTIALS = credentials('higher-privilege-service-account')  // Use the higher-privileged account
    }

    stages {
        stage('Authenticate with GCP') {
            steps {
                withCredentials([file(credentialsId: 'higher-privilege-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh 'gcloud config set project $GCR_PROJECT_ID'
                }
            }
        }

        stage('Grant IAM Roles') {
            steps {
                script {
                    sh """
                    gcloud projects add-iam-policy-binding $GCR_PROJECT_ID \
                        --member="serviceAccount:spinnaker-front50-sa@banded-setting-448905-d6.iam.gserviceaccount.com" \
                        --role="roles/serviceusage.serviceUsageAdmin"

                    gcloud projects add-iam-policy-binding $GCR_PROJECT_ID \
                        --member="serviceAccount:spinnaker-front50-sa@banded-setting-448905-d6.iam.gserviceaccount.com" \
                        --role="roles/storage.objectAdmin"
                    """
                }
            }
        }

        stage('Push Docker Image to GCR') {
            steps {
                script {
                    sh "docker tag $DOCKER_IMAGE $GCR_REPO:$IMAGE_TAG"
                    sh "docker push $GCR_REPO:$IMAGE_TAG"
                }
            }
        }
    }
}
