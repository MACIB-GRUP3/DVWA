pipeline {
    agent any

    // 🌎 Variables de Entorno
    environment {
        // Nombre de la configuración del servidor SonarQube en Jenkins
        SONARQUBE_SERVER = 'SonarQube' 
        // Parámetros de análisis de SonarQube
        SONAR_PROJECT_KEY = 'testPipeLine'
        SONAR_SOURCES = 'vulnerabilities'
        SONAR_PHP_VERSION = '8.0'
    }

    // 🏭 Etapas del Pipeline
    stages {
        stage('Checkout SCM') {
            steps {
                echo 'Checking out source code...'
                // Obtiene el código fuente de Git
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Starting SonarQube analysis for project: ${env.SONAR_PROJECT_KEY}"
                // Inyecta las credenciales y URL del servidor SonarQube
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh """
                    # 'sonar-scanner' ahora se encuentra en el PATH gracias al bloque 'tools'
                    sonar-scanner \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.sources=${SONAR_SOURCES} \
                    -Dsonar.php.version=${SONAR_PHP_VERSION}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate status...'
                // Espera hasta 10 minutos por el resultado del análisis
                timeout(time: 10, unit: 'MINUTES') {
                    // Falla el pipeline si el Quality Gate no pasa
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    // 🔔 Acciones Post-construcción
    post {
        always {
            echo 'Finalizando el pipeline.'
        }
        failure {
            echo 'El pipeline ha fallado. Revisar la etapa "SonarQube Analysis" o "Quality Gate".'
        }
        success {
            echo 'Pipeline ejecutado con éxito y Quality Gate aprobado.'
        }
    }
}
