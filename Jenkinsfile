pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        GITHUB_REPO = 'https://github.com/rahman13301/Nginx-Proxy-Manager.git'
        REPO_NAME = 'Nginx-Proxy-Manager'
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    echo 'Cloning repository...'
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        userRemoteConfigs: [[
                            url: env.GITHUB_REPO,
                            credentialsId: '' // Add your credentials ID if needed
                        ]],
                        extensions: [[
                            $class: 'CleanBeforeCheckout'
                        ]]
                    ])
                }
            }
        }
        
        stage('Validate Docker Compose') {
            steps {
                script {
                    echo 'Validating docker-compose file...'
                    sh '''
                        if [ -f "${DOCKER_COMPOSE_FILE}" ]; then
                            echo "docker-compose.yml found"
                            docker-compose config
                        else
                            echo "docker-compose.yml not found, checking for alternative names..."
                            find . -name "docker-compose*.yml" -o -name "docker-compose*.yaml"
                        fi
                    '''
                }
            }
        }
        
        stage('Build and Deploy') {
            steps {
                script {
                    echo 'Stopping existing containers...'
                    sh 'docker-compose down || true'
                    
                    echo 'Pulling latest images...'
                    sh 'docker-compose pull'
                    
                    echo 'Building and starting containers...'
                    sh 'docker-compose up -d --build'
                    
                    echo 'Checking container status...'
                    sh '''
                        sleep 10
                        docker-compose ps
                        docker-compose logs --tail=20
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    echo 'Performing health check...'
                    sh '''
                        # Check if containers are running
                        if docker-compose ps | grep -q "Up"; then
                            echo "Containers are running successfully"
                            
                            # You can add curl checks here for Nginx Proxy Manager
                            # curl -f http://localhost:81 || echo "Service check failed but continuing"
                        else
                            echo "Warning: Some containers may not be running"
                            docker-compose ps
                            exit 1
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Nginx Proxy Manager deployment completed successfully!'
            sh '''
                echo "Containers:"
                docker-compose ps
                echo "\nLogs (last 10 lines):"
                docker-compose logs --tail=10
            '''
        }
        failure {
            echo 'Deployment failed!'
            sh '''
                echo "Error logs:"
                docker-compose logs
                echo "\nContainer status:"
                docker-compose ps
            '''
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
