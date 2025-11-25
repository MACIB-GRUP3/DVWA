/**
 * Pipeline para la integración continua, check-out, análisis de código con SonarQube
 * y espera del Quality Gate.
 */
pipeline {
    // Define dónde se ejecutará el pipeline. 'any' significa que puede ejecutarse en cualquier agente.
    agent any

    // 🔨 Configuración de herramientas
    // Esto requiere que hayas configurado 'SonarQube Scanner' en "Manage Jenkins" -> "Global Tool Configuration"
    // (Ejemplo: le hemos dado el nombre 'SONAR_SCANNER_HOME' a la instalación).

    // 🌎 Variables de entorno
    environment {
        // Asegúrate de que 'SonarQube' coincide con el nombre de tu configuración de SonarQube en Jenkins
        SONARQUBE_SERVER = 'SonarQube'
        // Puedes definir otras variables para Sonar aquí si son constantes
        SONAR_PROJECT_KEY = 'testPipeLine'
        SONAR_SOURCES = 'vulnerabilities'
        SONAR_PHP_VERSION = '8.0'
        
        // La línea PATH original ya no es necesaria si usas el bloque 'tools'
        // PATH = "/opt/sonar-scanner/bin:${env.PATH}"
    }

    // 🏭 Definición de las etapas del Pipeline
    stages {
        stage('Checkout SCM') {
            steps {
                echo 'Checking out source code...'
                // Obtiene el código fuente de Git (basado en la configuración del Job)
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Starting SonarQube analysis for project: ${env.SONAR_PROJECT_KEY}"
                // Inyecta las variables de entorno para la conexión con SonarQube
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh """
                    # La herramienta 'sonar-scanner' está ahora en el PATH gracias al bloque 'tools'
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
                // Espera hasta 10 minutos por la respuesta del Quality Gate de SonarQube
                timeout(time: 10, unit: 'MINUTES') {
                    // 'abortPipeline: true' asegura que el pipeline falla si el Quality Gate no pasa
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    // 🔔 Acciones post-construcción (se ejecutan sin importar el resultado del pipeline)
    post {
        always {
            echo 'Finalizando el pipeline.'
        }
        failure {
            echo 'El pipeline ha fallado. Revisar la etapa "SonarQube Analysis".'
        }
        success {
            echo 'Pipeline ejecutado con éxito y Quality Gate aprobado.'
        }
    }
}
