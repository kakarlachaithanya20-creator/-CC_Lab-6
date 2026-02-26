pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Clean old containers & network
                docker rm -f backend1 backend2 nginx-lb || true
                docker network rm app-network || true

                # Create fresh network
                docker network create app-network

                # Start backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Wait for backend to fully initialize
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Start nginx on same network
                docker run -d --name nginx-lb \
                --network app-network \
                -p 80:80 nginx

                # Wait for nginx container
                sleep 3

                # Copy config
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx
                docker exec nginx-lb nginx -s reload

                # Wait a bit
                sleep 3
                '''
            }
        }

        stage('Verify Connectivity') {
            steps {
                sh '''
                echo "Testing connectivity from nginx to backend1..."
                docker exec nginx-lb wget -qO- backend1:8080 || true

                echo "Testing connectivity from nginx to backend2..."
                docker exec nginx-lb wget -qO- backend2:8080 || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. Load balancer should be working.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
