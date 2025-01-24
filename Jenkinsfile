pipeline { 

    agent any 

    environment { 

        SONARQUBE_SERVER = 'SQ'  // Name of your SonarQube server in Jenkins 

        DOCKER_IMAGE = 'helloworld-app:latest' // Replace with your desired image name

        IMAGE_TAG = 'latest' 

        IMAGE_NAME = 'helloworld-app' // Image name 
 

    } 

    stages { 

         
          stage('SCM') {
            checkout scm
          }
          
          stage('SonarQube Analysis') {
            def scannerHome = tool 'SonarScanner';
            withSonarQubeEnv(SONARQUBE_SERVER) {
              sh "${scannerHome}/bin/sonar-scanner"
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
       

