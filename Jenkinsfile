
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = "payment-api"
        IMAGE_NAME = "payment-api"
        CONTAINER_NAME = "payment-api-test"
        HOST_PORT = "8081"
        CONTAINER_PORT = "8080"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm

                script {
                    env.SHORT_GIT_COMMIT = bat(
                        script: '@git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Git Commit: ${env.GIT_COMMIT}"
                    echo "Short Commit: ${env.SHORT_GIT_COMMIT}"
                    echo "Build: ${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .
                    if errorlevel 1 exit /b 1

                    docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                    if errorlevel 1 exit /b 1

                    docker images %IMAGE_NAME%
                '''
            }
        }

        stage('Run Container') {
            steps {
                bat '''
                    docker rm -f %CONTAINER_NAME% 2>NUL || echo No existing test container

                    docker run -d ^
                        --name %CONTAINER_NAME% ^
                        -p %HOST_PORT%:%CONTAINER_PORT% ^
                        %IMAGE_NAME%:%BUILD_NUMBER%

                    if errorlevel 1 exit /b 1

                    powershell -NoProfile -Command "Start-Sleep -Seconds 5"

                    docker ps --filter "name=%CONTAINER_NAME%"
                '''
            }
        }

        stage('Troubleshoot Container') {
            steps {
                bat '''
                    echo ==============================
                    echo DOCKER PS
                    echo ==============================
                    docker ps -a --filter "name=%CONTAINER_NAME%"

                    echo ==============================
                    echo DOCKER PORT
                    echo ==============================
                    docker port %CONTAINER_NAME%

                    echo ==============================
                    echo DOCKER INSPECT
                    echo ==============================
                    docker inspect %CONTAINER_NAME%

                    echo ==============================
                    echo DOCKER TOP
                    echo ==============================
                    docker top %CONTAINER_NAME%

                    echo ==============================
                    echo DOCKER LOGS
                    echo ==============================
                    docker logs %CONTAINER_NAME%

                    echo ==============================
                    echo DOCKER EXEC
                    echo ==============================
                    docker exec %CONTAINER_NAME% whoami
                    if errorlevel 1 exit /b 1
                '''
            }
        }

        stage('Health Check') {
            steps {
                bat '''
                    echo Checking application health...

                    curl.exe --fail http://localhost:%HOST_PORT%/health
                    if errorlevel 1 (
                        echo Health check FAILED.
                        docker logs %CONTAINER_NAME%
                        exit /b 1
                    )

                    echo.
                    echo Health check PASSED.
                '''
            }
        }

        stage('Application Test') {
            steps {
                bat '''
                    echo Testing application endpoint...

                    curl.exe --fail http://localhost:%HOST_PORT%/
                    if errorlevel 1 (
                        echo Application test FAILED.
                        docker logs %CONTAINER_NAME%
                        exit /b 1
                    )

                    echo.
                    echo Application test PASSED.
                '''
            }
        }

        stage('Container Health Status') {
            steps {
                bat '''
                    echo Container health status:

                    docker inspect %CONTAINER_NAME% --format "{{.State.Health.Status}}"

                    docker inspect %CONTAINER_NAME% --format "{{.State.Health.Status}}" | findstr /I "healthy"
                    if errorlevel 1 (
                        echo Docker HEALTHCHECK is not healthy.
                        docker logs %CONTAINER_NAME%
                        exit /b 1
                    )

                    echo Docker HEALTHCHECK PASSED.
                '''
            }
        }
    }

    post {

        success {
            echo "======================================"
            echo "Lab-2 Docker assessment PASSED"
            echo "======================================"
            echo "Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
            echo "Build: ${BUILD_NUMBER}"
            echo "Commit: ${GIT_COMMIT}"
        }

        failure {
            echo "======================================"
            echo "Lab-2 Docker assessment FAILED"
            echo "======================================"
        }

        always {
            bat '''
                echo ==============================
                echo FINAL CONTAINER STATUS
                echo ==============================
                docker ps -a

                echo ==============================
                echo FINAL IMAGE
                echo ==============================
                docker images payment-api

                docker rm -f %CONTAINER_NAME% 2>NUL || echo Test container already removed
            '''
        }
    }
}

