pipeline {
    agent any

    parameters {
        choice(name: 'Backend_Count', choices: ['1', '2'], description: 'Select the number of backend instances to deploy')
    }

    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building Docker image..."
                docker rmi -f backend-app || true
                docker build -t backend-app ./backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Creating network and cleaning old containers..."
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                if [ "${Backend_Count}" = "1" ]; then
                    echo "Deploying a single backend instance (backend1)..."
                    docker run -d --name backend1 --network app-network backend-app
                else
                    echo "Deploying two backend instances (backend1 and backend2)..."
                    docker run -d --name backend1 --network app-network backend-app
                    docker run -d --name backend2 --network app-network backend-app
                fi
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Deploying NGINX Load Balancer..."
                docker rm -f nginx-lb || true
                
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            // Fixed: Removed the ${USER} variable that was causing the crash
            echo "Pipeline executed successfully. NGINX is balancing ${Backend_Count} container(s)."
        }
        failure {
            echo 'Pipeline failed. Check the console logs for specific error details.'
        }
    }
}
