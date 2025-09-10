@Library('shared-library') _

pipeline {
    agent any

    stages {
        
        stage('Clean up the Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Cloning the code from github') {
            steps {
                script{
                    clone('https://github.com/DattaRahegaonkar/chattingo-update.git','main')
                }
            }
        }

        stage('Creating the .env files for frontend and backend') {
            steps {
                withCredentials([
                    string(credentialsId: 'SPRING_DATASOURCE_USERNAME_ID', variable: 'SPRING_DATASOURCE_USERNAME'),
                    string(credentialsId: 'MYSQL_ROOT_PASSWORD_ID', variable: 'MYSQL_ROOT_PASSWORD'),
                    string(credentialsId: 'MYSQL_DATABASE_ID', variable: 'MYSQL_DATABASE'),
                    string(credentialsId: 'JWT_SECRET_ID', variable: 'JWT_SECRET'),
                    string(credentialsId: 'IP_ID', variable: 'IP'),
                    string(credentialsId: 'MYSQL_PASSWORD_ID', variable: 'MYSQL_PASSWORD')
                    
                ]) {
                    sh '''
                    echo ">>> Setting permissions..."
                    chmod -R 777 backend frontend

                    echo ">>> Creating backend .env..."
                    cat <<EOL | tee backend/.env
MYSQL_PASSWORD=$MYSQL_PASSWORD
MYSQL_USER=$SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_USERNAME=$SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_PASSWORD=$MYSQL_PASSWORD
SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/$MYSQL_DATABASE?createDatabaseIfNotExist=true
SPRING_PROFILES_ACTIVE=production
SERVER_PORT=8081
JWT_SECRET=$JWT_SECRET
# CORS_ALLOWED_ORIGINS=http://$IP:3000
CORS_ALLOWED_ORIGINS=http://chattingo.rahegaonkar.site,https://chattingo.rahegaonkar.site,http://localhost:3000,http://127.0.0.1:3000
CORS_ALLOWED_METHODS=GET,POST,PUT,DELETE,OPTIONS,PATCH
CORS_ALLOWED_HEADERS=*
WEBSOCKET_ENDPOINT=/ws
MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD
MYSQL_DATABASE=$MYSQL_DATABASE
EOL

                    echo ">>> Creating frontend .env..."
                    cat <<EOL | tee frontend/.env
# REACT_APP_API_URL=http://$IP:8081
# REACT_APP_WS_URL=http://$IP:8081/ws
REACT_APP_API_URL=https://chattingo.rahegaonkar.site
REACT_APP_WS_URL=https://chattingo.rahegaonkar.site
REACT_APP_ENV=production
REACT_APP_DEBUG=true
EOL

                    echo ">>> Fixing ownership..."
                    chown -R jenkins:jenkins backend frontend
                    '''
                }
            }
        }
        
        stage("OWASP Dependency Cheack") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: "**/dependency-check-report.xml"
            }
        }
        
         stage("Scanning the File Syatem using Trivy") {
            steps {
                script{
                    trivyscan("chattingo-file-system-scan-report-${env.BUILD_NUMBER}.html")
                }
            }
        }
        
        stage("Building the Backend and Frontend Docker Images") {
            steps {
                script{
                    dockerbuild("hackthon-chattingo-app_backend","latest", 'backend')
                    dockerbuild("hackthon-chattingo-app_frontend","latest", 'frontend')
                }
            }
        }
        
        stage("Images Scanning Using Trivy") {
            steps {
                script{
                    trivyimagescan("hackthon-chattingo-app_backend","backend-image-scan-report-${env.BUILD_NUMBER}.html")
                    trivyimagescan("hackthon-chattingo-app_frontend","frontend-image-scan-report-${env.BUILD_NUMBER}.html")
                }
            }
        }
        
        stage("Pushing the Images on Docker Hub Repository") {
            steps {
                script{
                    dockerpush("docker_hub_creds","hackthon-chattingo-app_backend", "${env.BUILD_NUMBER}")
                    dockerpush("docker_hub_creds","hackthon-chattingo-app_frontend","${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Update docker-compose file') {
            steps {
                script {
                    sh """
                    sed -i 's|dattarahegaonkar09/hackthon-chattingo-app_backend:[^ ]*|dattarahegaonkar09/hackthon-chattingo-app_backend:${env.BUILD_NUMBER}|g' docker-compose.yml
                    sed -i 's|dattarahegaonkar09/hackthon-chattingo-app_frontend:[^ ]*|dattarahegaonkar09/hackthon-chattingo-app_frontend:${env.BUILD_NUMBER}|g' docker-compose.yml
                    """
                }
            }
        }

        stage('Deploying using Docker Compose') {
            steps {
                sh '''
                echo ">>> Removing old containers"
                docker rm -f mysql backend frontend chattingo-nginx chattingo-certbot || true
                docker compose down || true

                echo ">>> Creating new containers"
                # docker-compose build --no-cache
                docker compose up -d
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "**/*.html, **/*.xml", allowEmptyArchive: true
        }
    }
}

