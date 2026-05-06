pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'sonarqube-server'
        PROJECT_KEY = 'fastapi-sonar-project'
    }

    stages {

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'python:3.11'
                    args '-u root'
                }
            }
            steps {
                sh '''
                python -m pip install --upgrade pip

                if [ -f requirements.txt ]; then
                    pip install -r requirements.txt
                fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                    args '-u root'   // ✅ CRITICAL FIX
                }
            }
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=fastapi-sonar-project \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN \
                      -Dsonar.userHome=.sonar
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
