pipeline {
    // Agente principal para etapas que no requieren sonar-scanner (como checkout y quality gate)
    agent any 

    // 🌎 Variables de Entorno
    environment {
        // Asegúrate de que 'SonarQube' coincide con el nombre de tu configuración en Jenkins
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
                // Obtiene el código fuente de Git en el workspace del agente
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            // 🐳 CAMBIO CLAVE: Cambia el contexto de ejecución a un contenedor Docker.
            agent {
                docker {
                    // Usamos la imagen oficial que ya tiene 'sonar-scanner' instalado.
                    image 'sonarsource/sonar-scanner-cli' 
                    // Mapeamos el directorio de trabajo del agente al contenedor ($PWD) 
                    // para que el escáner pueda acceder a los archivos fuente.
                    args '-v $PWD:$PWD -w $PWD' 
                }
            }
            steps {
                echo "Starting SonarQube analysis for project: ${env.SONAR_PROJECT_KEY}"
                // Inyecta las credenciales y URL del servidor SonarQube
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh """
                    # El comando 'sonar-scanner' funciona ahora porque está en la imagen Docker.
                    sonar-scanner \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.sources=${SONAR_SOURCES} \
                    -Dsonar.php.version=${SONAR_PHP_VERSION}
                    """
                }
            }
        }

        stage('Quality Gate') {
            // Volvemos al agente principal (o sigue usando 'agent none' si quieres que Jenkins lo decida)
            agent any 
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
