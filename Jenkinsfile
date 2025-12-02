pipeline {
    agent any
    
    environment {
        COMPOSE_PROJECT_NAME = 'nginx-proxy-manager-${BUILD_NUMBER}'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/rahman13301/Nginx-Proxy-Manager.git'
                    ]]
                ])
            }
        }
        
        stage('Cleanup Existing Containers') {
            steps {
                script {
                    sh '''
                        echo "Cleaning up any existing containers and networks..."
                        
                        # Remove any container named npm
                        docker rm -f npm 2>/dev/null || true
                        
                        # Stop and remove any containers from previous builds
                        docker ps -a --filter "name=npm" --format "{{.ID}}" | xargs -r docker rm -f
                        
                        # Clean up networks (be careful with this in shared environments)
                        docker network prune -f 2>/dev/null || true
                        
                        # Remove orphaned containers
                        docker-compose down --remove-orphans 2>/dev/null || true
                        
                        echo "Cleanup completed"
                    '''
                }
            }
        }
        
        stage('Prepare Environment') {
            steps {
                script {
                    // Check if docker-compose.yml exists
                    if (fileExists('docker-compose.yml')) {
                        echo 'Found existing docker-compose.yml, backing it up...'
                        sh 'mv docker-compose.yml docker-compose.yml.backup'
                    }
                    
                    // Create fresh docker-compose.yml
                    writeFile file: 'docker-compose.yml', text: '''version: '3.8'
services:
  npm:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: npm-${BUILD_NUMBER}
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
      DISABLE_IPV6: "true"
    networks:
      - npm-network

networks:
  npm-network:
    driver: bridge
'''
                    
                    echo 'Created docker-compose.yml with unique container name'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sh '''
                        echo "Current directory:"
                        pwd
                        echo ""
                        echo "Files in directory:"
                        ls -la
                        echo ""
                        echo "Docker-compose.yml content:"
                        cat docker-compose.yml
                        echo ""
                        
                        # Create directories
                        mkdir -p data letsencrypt
                        
                        # Set unique project name
                        export COMPOSE_PROJECT_NAME="npm-${BUILD_NUMBER}"
                        
                        echo "Using project name: ${COMPOSE_PROJECT_NAME}"
                        
                        # Check for duplicate docker-compose files
                        echo "Checking for duplicate compose files..."
                        find . -name "docker-compose*" -o -name "compose*" | while read file; do
                            if [ "$file" != "./docker-compose.yml" ]; then
                                echo "Removing duplicate: $file"
                                rm -f "$file"
                            fi
                        done
                        
                        # Verify only one docker-compose.yml exists
                        echo "Remaining compose files:"
                        find . -name "docker-compose*" -o -name "compose*" 2>/dev/null || true
                        
                        # Deploy with explicit file specification
                        docker-compose -f docker-compose.yml down --remove-orphans 2>/dev/null || true
                        
                        echo "Starting Nginx Proxy Manager..."
                        docker-compose -f docker-compose.yml up -d
                        
                        # Wait for startup
                        sleep 20
                    '''
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sh '''
                        echo "=== Container Status ==="
                        docker-compose -f docker-compose.yml ps
                        
                        echo ""
                        echo "=== Docker Container List ==="
                        docker ps --filter "name=npm" --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"
                        
                        echo ""
                        echo "=== Checking Service Health ==="
                        
                        # Check if container is running
                        CONTAINER_ID=$(docker ps -q --filter "name=npm-${BUILD_NUMBER}")
                        if [ -z "$CONTAINER_ID" ]; then
                            echo "ERROR: Container is not running!"
                            echo "Checking logs..."
                            docker-compose -f docker-compose.yml logs
                            exit 1
                        fi
                        
                        echo "Container ID: ${CONTAINER_ID}"
                        echo "Container is running"
                        
                        # Check logs for errors
                        echo ""
                        echo "=== Recent Logs ==="
                        docker-compose -f docker-compose.yml logs --tail=20
                        
                        # Simple port check
                        echo ""
                        echo "=== Port Check ==="
                        if timeout 5 bash -c 'cat < /dev/null > /dev/tcp/localhost/81' 2>/dev/null; then
                            echo "✓ Port 81 is listening"
                        else
                            echo "⚠ Port 81 is not accessible (may still be starting)"
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
            script {
                sh '''
                    echo "=========================================="
                    echo "NGINX PROXY MANAGER DEPLOYED SUCCESSFULLY"
                    echo "=========================================="
                    echo ""
                    echo "Access the management interface at:"
                    echo "http://YOUR_SERVER_IP:81"
                    echo ""
                    echo "Default credentials:"
                    echo "Email:    admin@example.com"
                    echo "Password: changeme"
                    echo ""
                    echo "Container name: npm-${BUILD_NUMBER}"
                    echo "To view logs: docker logs npm-${BUILD_NUMBER}"
                    echo "To stop: docker-compose -f docker-compose.yml down"
                    echo "=========================================="
                '''
            }
        }
        
        failure {
            echo 'Deployment failed!'
            script {
                sh '''
                    echo "=== DEBUG INFORMATION ==="
                    echo ""
                    echo "Docker containers:"
                    docker ps -a
                    echo ""
                    echo "Docker networks:"
                    docker network ls
                    echo ""
                    echo "Full logs:"
                    docker-compose -f docker-compose.yml logs 2>/dev/null || echo "Could not get logs"
                    echo ""
                    echo "Directory contents:"
                    ls -la
                    echo ""
                    echo "Removing failed containers..."
                    docker-compose -f docker-compose.yml down 2>/dev/null || true
                '''
            }
        }
        
        always {
            echo 'Build completed'
            // Optional: Archive docker-compose.yml
            archiveArtifacts artifacts: 'docker-compose.yml'
        }
    }
}
