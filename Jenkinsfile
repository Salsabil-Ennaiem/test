pipeline {
    agent any
    
    options { timestamps() }

    environment {
        // CHANGE mohamed123 to YOUR Docker Hub username !!!
        IMAGE = 'mohamed123/monapp'
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
            }
        }

        stage('Smoke Test') {
            steps {
                bat """
                    docker rm -f monapp_test 2>nul || ver > nul
                    docker run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
                    timeout /t 5
                    curl -I http://localhost:8088 | findstr "200 OK"
                    if %errorlevel% neq 0 exit 1
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
            }
        }

        stage('Cleanup Local Images') {
            steps {
                bat """
                    docker rmi %IMAGE%:%TAG%
                    docker rmi %IMAGE%:latest
                    docker system prune -f
                """
            }
        }
    }

    post {
        success {
            echo '🎉 Build + Test + Push SUCCESS!'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}