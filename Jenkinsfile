pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm: [
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/rahman13301/Nginx-Proxy-Manager.git'
                    ]]
                ]
                
                script {
                    // Create docker-compose.yml if it doesn't exist
                    if (!fileExists('docker-compose.yml')) {
                        writeFile file: 'docker-compose.yml', text: '''
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
'''
                        echo 'Created docker-compose.yml file'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    # Create directories
                    mkdir -p data letsencrypt
                    
                    # Deploy using docker-compose
                    docker-compose down || true
                    docker-compose up -d
                    
                    # Check status
                    sleep 15
                    docker-compose ps
                '''
            }
        }
    }
}
