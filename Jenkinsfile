pipeline {
    agent any

    environment {
        APP_NAME = 'rcat-project'
        DOCKER_IMAGE = 'aditya20266/rcat-project'
        CONTAINER_NAME = 'rcat-project'
        HOST_PORT = '8081'
        CONTAINER_PORT = '8080'
    }

    stages {

        stage('Environment Check') {
            steps {
                sh '''
                    echo "=== Jenkins Environment ==="
                    java -version || true
                    mvn -version || true
                    docker --version
                '''
            }
        }

        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/adityachaurasiya0112-code/rcat-project.git',
                    branch: 'main',
                    credentialsId: 'github'
                )
            }
        }

        stage('Build with Java 17') {
            steps {
                sh '''
                    set -e

                    echo "=== Building project with Java 17 ==="

                    docker run --rm \
                        -v "$WORKSPACE:/workspace" \
                        -w /workspace \
                        maven:3.9.11-eclipse-temurin-17 \
                        mvn clean package -DskipTests

                    echo "Build completed successfully."
                '''
            }
        }

        stage('Test with Java 17') {
            steps {
                sh '''
                    set -e

                    echo "=== Running tests with Java 17 ==="

                    docker run --rm \
                        -v "$WORKSPACE:/workspace" \
                        -w /workspace \
                        maven:3.9.11-eclipse-temurin-17 \
                        mvn test

                    echo "Tests completed successfully."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "=== Building Docker Image ==="

                    docker build \
                        -t ${DOCKER_IMAGE}:latest \
                        .

                    echo "Docker image built successfully."
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -e

                        echo "=== Logging into Docker Hub ==="

                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${DOCKER_IMAGE}:latest

                        docker logout

                        echo "Docker image pushed successfully."
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e

                    echo "=== Deploying Application ==="

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${HOST_PORT}:${CONTAINER_PORT} \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:latest

                    sleep 10

                    docker ps --filter "name=${CONTAINER_NAME}"
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "=== Health Check ==="

                    if curl -f http://localhost:${HOST_PORT}/; then
                        echo "Application is UP!"
                    else
                        echo "Application health check failed."
                        docker logs ${CONTAINER_NAME}
                        exit 1
                    fi
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the logs above.'
        }

        always {
            sh '''
                docker ps -a --filter "name=rcat-project" || true
            '''
        }
    }
}
