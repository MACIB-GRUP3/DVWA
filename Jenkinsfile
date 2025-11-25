pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
        PATH = "/opt/sonar-scanner/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=testPipeLine \
                        -Dsonar.sources=vulnerabilities \
                        -Dsonar.php.version=8.0
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
