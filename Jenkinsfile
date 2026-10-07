```groovy
pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        JAVA17_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'

        // Docker
        DOCKER_IMAGE = 'aditya20266/rcat-project'
        CONTAINER_NAME = 'rcat-project'

        APP_PORT = '8081'
        CONTAINER_PORT = '8080'

        // SonarQube
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

                    echo "===== Java 17 ====="
                    ${JAVA17_HOME}/bin/java -version

                    echo "===== Maven ====="
                    export JAVA_HOME=${JAVA17_HOME}
                    export PATH=${JAVA_HOME}/bin:/usr/bin:/bin
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

                    export JAVA_HOME=${JAVA17_HOME}
                    export PATH=${JAVA_HOME}/bin:/usr/bin:/bin

                    echo "========================================"
                    echo "          MAVEN BUILD"
                    echo "========================================"

                    echo "JAVA_HOME=${JAVA_HOME}"

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

                    export JAVA_HOME=${JAVA17_HOME}
                    export PATH=${JAVA_HOME}/bin:/usr/bin:/bin

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

                        export JAVA_HOME=${JAVA17_HOME}
                        export PATH=${JAVA_HOME}/bin:/usr/bin:/bin

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
```
