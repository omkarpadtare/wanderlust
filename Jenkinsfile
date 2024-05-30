pipeline {
    agent any

        environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }
    
    stages {  
        // stage('pullcode') {  
        //     steps {  
        //         // git credentialsId: '99849407-bbec-4b89-91e5-41dda1989b57', url: 'https://github.com/omkarpadtare/nodejs.git'
        //     }  
        // }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    // Build and push Docker images defined in docker-compose.yml
                    def services = ['backend', 'frontend'] // List your services here
                    for (service in services) {
                        def image = "omkarphadtare321/nodejs-applicaiton_${GIT_BRANCH}-${service}"
                        def tag = 'latest'
                        
                        // Build the Docker image
                        sh "docker compose -f ${DOCKER_COMPOSE_FILE} build ${service}"
                        
                        // Tag the Docker image
                        sh "docker tag nodejs-applicaiton_${GIT_BRANCH}-${service}:latest ${image}:${tag}"
                        
                        // Push  the Docker image to Docker Hub
                        // docker.withRegistry('https://registry.hub.docker.com/', 'docker-cred') {
                        //     sh "docker push ${image}:${tag}"
                        // }
                    }
                }
            }
        }

  stage('SonarQube analysis') {
            environment {
                SCANNER_HOME = tool 'sonarqube';    
            }
            
            steps {
                
                withSonarQubeEnv('sonarqube') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Deploy') {
            steps {
                // Use Docker Compose to deploy the services
                sh 'docker compose down'
                sh 'docker compose up -d'
                sh 'sudo docker exec mongo mongoimport --db wanderlust --collection posts --file ./data/sample_posts.json --jsonArray'
            }
        }
    }

       post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Clean up resources, send notifications, etc.
            echo 'This will always run'
        }
    }
}
