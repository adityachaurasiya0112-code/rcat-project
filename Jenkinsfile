pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKER_IMAGE = 'aditya20266/rcat-project'
        CONTAINER_NAME = 'rcat-project'

        APP_PORT = '8081'
        CONTAINER_PORT = '8080'

        SONAR_PLUGIN = 'org.sonarsource.scanner.maven:sonar-maven-plugin:5.4.0.6343'
    }

    stages {

        stage('Environment Check') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "       ENVIRONMENT CHECK"
                    echo "========================================"

                    echo "===== Java ====="
                    java -version

                    echo "===== Java Path ====="
                    which java

                    echo "===== JAVA_HOME ====="
                    echo "${JAVA_HOME:-JAVA_HOME is not set}"

                    echo "===== Maven ====="
                    mvn -version

                    echo "===== Docker ====="
                    docker --version

                    echo "========================================"
                '''
            }
        }

        stage('Checkout') {
            steps {
                echo "===== Checking Out GitHub Repository ====="

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

                    echo "========================================"
                    echo "          MAVEN BUILD"
                    echo "========================================"

                    java -version
                    mvn -version

                    mvn clean package -DskipTests

                    echo "===== Maven Build Successful ====="
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "             TESTS"
                    echo "========================================"

                    mvn test

                    echo "===== Tests Passed ====="
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        set -e

                        echo "========================================"
                        echo "       SONARQUBE ANALYSIS"
                        echo "========================================"

                        echo "SonarQube Server: ${SONAR_HOST_URL}"

                        java -version
                        mvn -version

                        mvn ${SONAR_PLUGIN}:sonar \
                            -Dsonar.projectKey=rcat-project \
                            -Dsonar.projectName=rcat-project \
                            -Dsonar.host.url=${SONAR_HOST_URL}

                        echo "===== SonarQube Analysis Completed ====="
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "========================================"
                echo "       SONARQUBE QUALITY GATE"
                echo "========================================"

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }

                echo "===== Quality Gate Passed ====="
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "          DOCKER BUILD"
                    echo "========================================"

                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -t ${DOCKER_IMAGE}:latest \
                        .

                    echo "===== Docker Image Built Successfully ====="

                    docker images ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Docker Hub Login') {
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

                        echo "===== Logging in to Docker Hub ====="

                        echo "${DOCKER_PASSWORD}" | docker login \
                            -u "${DOCKER_USERNAME}" \
                            --password-stdin

                        echo "===== Docker Hub Login Successful ====="
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "          DOCKER PUSH"
                    echo "========================================"

                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest

                    echo "===== Docker Images Pushed Successfully ====="
                '''
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "         DOCKER DEPLOY"
                    echo "========================================"

                    echo "===== Removing Old Container ====="

                    docker rm -f ${CONTAINER_NAME} 2>/dev/null || true

                    echo "===== Starting New Container ====="

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:${CONTAINER_PORT} \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    echo "===== Container Started ====="

                    sleep 10

                    echo "===== Container Status ====="

                    docker ps \
                        --filter "name=${CONTAINER_NAME}"

                    echo "===== Application Health Check ====="

                    if curl -f --max-time 10 http://localhost:${APP_PORT}; then
                        echo "===== Application is UP ====="
                    else
                        echo "===== Application Health Check FAILED ====="

                        echo "===== Docker Logs ====="

                        docker logs ${CONTAINER_NAME} --tail 100

                        exit 1
                    fi

                    echo "===== Deployment Successful ====="
                '''
            }
        }
    }

    post {

        success {
            echo '''
========================================
       PIPELINE SUCCESSFUL
========================================

Application:
http://65.2.56.162:8081

Jenkins:
http://13.207.56.243:8080

SonarQube:
http://13.207.56.243:9000

Docker Image:
aditya20266/rcat-project:${BUILD_NUMBER}

Docker Container:
rcat-project

========================================
'''
        }

        failure {
            echo '''
========================================
         PIPELINE FAILED
========================================
'''

            sh '''
                echo "===== Docker Containers ====="

                docker ps -a \
                    --filter "name=${CONTAINER_NAME}" || true

                echo "===== Docker Logs ====="

                docker logs ${CONTAINER_NAME} \
                    --tail 100 2>/dev/null || true
            '''
        }

        always {
            echo "Pipeline finished: ${currentBuild.currentResult}"
        }
    }
}

