pipeline {
    agent {
        docker {
            image 'python:3.11'
            args '-u root'
        }
    }

    environment {
        SONARQUBE_SERVER = 'sonarqube-server'
        PROJECT_KEY = 'fastapi-sonar-project'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Jyothiting/sonar-project-api.git'
            }
        }

        stage('Setup & Install Dependencies') {
            steps {
                sh '''
                python -m venv venv
                . venv/bin/activate

                pip install --upgrade pip

                if [ -f requirements.txt ]; then
                    pip install -r requirements.txt
                fi

                pip install sonar-scanner-cli
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh '''
                    . venv/bin/activate

                    sonar-scanner \
                      -Dsonar.projectKey=${PROJECT_KEY} \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
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
