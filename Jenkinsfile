pipeline { 

    agent any 

    environment { 

        SONARQUBE_SERVER = 'SQ'  // Name of your SonarQube server in Jenkins 

        DOCKER_IMAGE = 'helloworld-app:latest' // Replace with your desired image name

        IMAGE_TAG = 'latest' 

        IMAGE_NAME = 'helloworld-app' // Image name 
 

    } 

    stages { 

        stage('Git Checkout Stage') { 

            steps { 

              checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'Bharath13041999', url: 'https://github.com/Bharath13041999/java.git']]) 

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

         

        stage('Run Docker Container') { 

            steps { 

                sh 'dock
