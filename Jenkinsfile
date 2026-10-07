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

        stage('Setup Java 17') {
            steps {
                sh '''
                    set -e

                    echo "=== Setting up Java 17 ==="

                    if ! command -v java >/dev/null 2>&1 || \
                       ! java -version 2>&1 | grep -q '17\\.'; then

                        echo "Java 17 not found. Installing automatically..."

                        sudo apt-get update
                        sudo apt-get install -y openjdk-17-jdk

                    else
                        echo "Java 17 already installed."
                    fi

                    JAVA17_HOME=$(dirname $(dirname $(readlink -f $(which java))))

                    echo "JAVA17_HOME=$JAVA17_HOME"

                    export JAVA_HOME="$JAVA17_HOME"
                    export PATH="$JAVA_HOME/bin:$PATH"

                    java -version
                    javac -version

                    echo "Java 17 setup completed."
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

        stage('Build') {
            steps {
                sh '''
                    set -e

                    JAVA17_HOME=$(dirname $(dirname $(readlink -f $(which java))))

                    export JAVA_HOME="$JAVA17_HOME"
                    export PATH="$JAVA_HOME/bin:$PATH"

                    echo "Building with:"
                    java -version
                    mvn -version

                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e

                    JAVA17_HOME=$(dirname $(dirname $(readlink -f $(which java))))

                    export JAVA_HOME="$JAVA17_HOME"
                    export PATH="$JAVA_HOME/bin:$PATH"

                    mvn test
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:latest .
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
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
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
                    echo "Checking application..."

                    curl -f http://localhost:${HOST_PORT}/ || {
                        echo "Application health check failed"
                        docker logs ${CONTAINER_NAME}
                        exit 1
                    }

                    echo "Application is UP!"
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
            sh 'docker ps -a --filter "name=rcat-project" || true'
        }
    }
}
