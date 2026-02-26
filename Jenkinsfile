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
                # Clean old containers and network
                docker rm -f backend1 backend2 nginx-lb || true
                docker network rm app-network || true

                # Create fresh network
                docker network create app-network

                # Start backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Wait for backend servers to fully start
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Start nginx AFTER backend containers are running
                docker run -d --name nginx-lb --network app-network -p 80:80 nginx

                # Wait for nginx to initialize
                sleep 3

                # Copy configuration
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx
                docker exec nginx-lb nginx -s reload

                sleep 2
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
