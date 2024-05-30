pipeline {
    agent any

        environment {
        // Define environment variables
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        SONARQUBE_SCANNER_HOME = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
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
                        
                        // Push the Docker image to Docker Hub
                        // docker.withRegistry('https://registry.hub.docker.com/', 'docker-cred') {
                        //     sh "docker push ${image}:${tag}"
                        // }
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            environment {
                // Use the SonarQube token stored in Jenkins credentials
                SONAR_TOKEN = credentials('sonar-token')
            }
            steps {
                withSonarQubeEnv('SonarQube Server') {
                    sh '''
                        ${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=wanderlust \
                        -Dsonar.sources=./src \
                        -Dsonar.host.url=http://54.156.7.93:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
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
