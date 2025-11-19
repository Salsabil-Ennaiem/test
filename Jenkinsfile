pipeline {
    agent any
    
    options { 
        timestamps()
        skipDefaultCheckout(true) 
    }

    environment {
        IMAGE = 'oksee/monapp'
        TAG   = "build-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker version'
                bat "docker build -t %IMAGE%:%TAG% ."
                bat "docker tag %IMAGE%:%TAG% %IMAGE%:latest"
                bat "docker images %IMAGE%"   // just to see it in console
            }
        }

        stage('Smoke Test on port 8088') {
            steps {
                bat """
                    docker rm -f monapp_test 2>nul || ver > nul
                    docker run -d --name monapp_test -p 8088:80 %IMAGE%:%TAG%
                    timeout /t 8
                    curl -I http://localhost:8088 | findstr "200 OK"
                    if %errorlevel% neq 0 (
                        echo TEST FAILED - PORT 8088 NOT RESPONDING
                        exit 1
                    ) else (
                        echo TEST PASSED ON PORT 8088
                    )
                    docker rm -f monapp_test
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                              usernameVariable: 'USER', 
                                              passwordVariable: 'PASS')]) {
                    bat """
                        echo %PASS% | docker login -u %USER% --password-stdin
                        docker push %IMAGE%:%TAG%
                        docker push %IMAGE%:latest
                    """
                }
                echo "Images pushed → https://hub.docker.com/r/oksee/monapp"
            }
        }

        stage('Cleanup') {
            steps {
                bat """
                    docker rmi %IMAGE%:%TAG% 2>nul || ver > nul
                    docker rmi %IMAGE%:latest 2>nul || ver > nul
                    docker system prune -f
                """
            }
        }
    }

    post {
        success {
            echo 'SUCCESS ! Go check https://hub.docker.com/r/oksee/monapp → you will see build-X and latest'
        }
        failure {
            echo 'FAILED - check console output above'
        }
        always {
            cleanWs()
        }
    }
}